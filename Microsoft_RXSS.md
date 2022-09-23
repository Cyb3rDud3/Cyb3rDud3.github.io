---

layout: page
title: "Microsoft -MSN Weather forecast reflected XSS"
permalink: /BugBounty/MSN_WEATHER_RXSS
---



## **Microsoft -MSN Weather forecast reflected XSS**

On Jun 16, 2022 , while checking the weather at MSN, I found a suspicious base64 object in the "loc" parameter of the URL.

After some fuzzing, I found a stealthy(base64 encoded) reflected XSS In the URL.
the Vulnerability has ben reported to MSRC, and was finally patched on Aug 25,2022. 

***The problem:***
	
	 - the url: https://www.msn.com/he-il/weather/forecast/?loc=${payload_here}&weadegreetype=C It is designed to get a base64 encoded JSON Object that contains information about your current location, including city name, country name, latitude, language, and more.
	 - Example :{"l":CityName","r":"DistrictName","c":"CountryName","i":"CountryCode","g":"LangCode","x":float,"y": float}
	 - by putting simple XSS payload inside the object and encoding It as base64, we are closing the html tags.
	 - payload: {"l":"EVIL_CITY\" /> alert('EVIL')","r":"Center District","c":"Israel","i":"IL","g":"he-il","x":34.000000000000,"y":34.000000000000}

***The Patch:*** 
	 I didn't actually get any details about how Microsoft fixed this issue, but after searching through the source code, I found the change. 
	 I can't be sure that it's the full fix, but that's what I found.
		 

    Before:
	    const n = JSON.parse(e)
	After:
		const n = JSON.parse(e.replace(/[<>]/g, ''))
	assume variable "e" is the base64 decoded object from the url.

**POC video**: 


<video src="https://user-images.githubusercontent.com/50178939/191971350-2677f80c-03cd-42f7-915e-ffa361fad0f0.mp4" controls="controls" style="max-width: 730px;">
	</video>



[HackTheBox](https://app.hackthebox.com/profile/360735)

[Linkedin](https://www.linkedin.com/in/guy-h087/)

[Work Mail](mailto://cyb3rguy1337@gmail.com)
