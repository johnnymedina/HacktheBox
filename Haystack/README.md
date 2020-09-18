# Haystack
Haystack is a simple box with bit of a sticky point in finding and translating information through the mess. 

Enumeration -> Analysis -> Exploitation ->Post Exploitation 

### Enumeration
I started with an nmap scan of the systems to see what services were avaiable

```nmap 10.10.10.115 -T4 -sV -sC```

<p align="center"> 
<img src="https://raw.githubusercontent.com/johnnymedina/HacktheBox/master/Haystack/1-nmap.png" width="50%">
</p>

Next, I used dirb to see what was running on the webservices on port 80 and 9200 identified.

``dirb http://10.10.10.115:9200``

<p align="center"> 
<img src="https://raw.githubusercontent.com/johnnymedina/HacktheBox/master/Haystack/2-httpports.png" width="50%">
</p>

Dirb returned nothing on port 80, and returned too much noise in Code 400s on port 9200.
So I decided to visit each port through a web browser

<p align="center"> 
<img src="https://raw.githubusercontent.com/johnnymedina/HacktheBox/master/Haystack/3-port80.png" width="50%">
</p>

Browser on port 9200:

<p align="center"> 
<img src="https://raw.githubusercontent.com/johnnymedina/HacktheBox/master/Haystack/4-port9200.png" width="50%">
</p>

## Analysis
So we have a .jpg on port 80 and Elastic Search 6.4.2 on 9200. I decided to download the .jpeg and do some forensics on it with binwalk and just plain cat.

```binwalk needle.jpg```

<p align="center"> 
<img src="https://raw.githubusercontent.com/johnnymedina/HacktheBox/master/Haystack/5-forensics.png" width="50%">
</p>

There is a base 64 encoded string at the bottom of the cat-ed jpg which is odd.

```echo <string> | base64 -d```

<p align="center"> 
<img src="https://github.com/johnnymedina/HacktheBox/blob/master/Haystack/6-base64.png" width="50%">
</p>

I threw it into Google Translate to see what it said.
<p align="center"> 
<img src="https://raw.githubusercontent.com/johnnymedina/HacktheBox/master/Haystack/7-translate.pngg" width="50%">
</p>

Since we know its Elastic search on port 9200, I ran metasploit module to enumerate indices from Elastic search to avoid the noise given using dirb and get back 3 "bank, .kibana,quotes

<p align="center"> 
<img src="https://raw.githubusercontent.com/johnnymedina/HacktheBox/master/Haystack/8-elastic-enum.png" width="50%">
</p>

On Stack Overflow found quick way to retrieve all records from Elastic Search
https://stackoverflow.com/questions/8829468/elasticsearch-query-to-return-all-records

<p align="center"> 
<img src="https://raw.githubusercontent.com/johnnymedina/HacktheBox/master/Haystack/9-elestic-reserarch.png" width="50%">
</p>

I also decided to take a look at API documentation from Elastic website

<p align="center"> 
<img src="https://raw.githubusercontent.com/johnnymedina/HacktheBox/master/Haystack/10-elastic-api.png"width ="40%">
</p>

## Exploitation
With the information I gathered from the API documentation I was able to create a quick search string.
```http://10.10.10.115:9200/.kibana/_search?size=100&q=*:*```

<p align="center"> 
<img src="https://raw.githubusercontent.com/johnnymedina/HacktheBox/master/Haystack/11-rawoutput.png" width="50%">
</p>

Searched raw output for term "clave" and found base64 encoded strings, which then used google translate 

<p align="center"> 
<img src="https://raw.githubusercontent.com/johnnymedina/HacktheBox/master/Haystack/12-translate.png" width="50%">
</p>

Then used command 'base64 -d' to get decoded strings
```echo <String> | base64 -d```

<p align="center"> 
<img src="https://raw.githubusercontent.com/johnnymedina/HacktheBox/master/Haystack/13-base64-decode.png" width="50%">
</p>

<p align="center"> 
<img src="https://raw.githubusercontent.com/johnnymedina/HacktheBox/master/Haystack/14-base64-decode-key.png" width="50%">
</p>

From that we now have our user and password combo to try on another service like SSH that was open from the nmap results.

```ssh security@10.10.10.115```

<p align="center"> 
<img src="https://raw.githubusercontent.com/johnnymedina/HacktheBox/master/Haystack/15-ssh.png" width="50%">
</p>


## Post Exploitation
Then just run cat or strings to get the flag

```cat flag.txt```
