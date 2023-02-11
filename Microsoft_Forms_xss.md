
## 8 Lines of code, 3 different stored XSS bugs in Microsoft Forms

Let's explore the mystery of 3 different stored XSS vectors I found in the same 8 lines of client-side Javascript code used by Microsoft Forms.

   **Every one of those bugs resulted in stored XSS on every input in Microsoft Forms that allowed you to style your text, from question, title, description, thank you message - pretty much anywhere** 
   


https://user-images.githubusercontent.com/50178939/218260169-d010e729-ccd1-44a2-a508-2ff2709ecf11.mp4





    function r(n) {
	return !!n && function (n) {
	var e = /<\/?[^>]*>/i,
		t = /<\/?(strong|em|u|sup|sub|i|b|br|ul|li|span|ol)>/gi,
		r = /<(span|b|i|u)\s?((style=)('|")\s*('|"))>/gi,
		i = /(font-size:\s(1.2|1|0.8|0.9|1.3)em)(;)/gi,
		o = /color:\s(rgb)\(([^)]+)\)(;)/gi,
		u = n.replace(t, "").replace(i, "").replace(o, "");
		if (e.test(u)) return !e.test(u.replace(r, ""));
		return !0
	}(n)}

 *This is the first and the original version of the code. 
 
What does this code do? Unfortunately, The logic is not so easy to understand because it's minified, and I didn't include the function that calls this function.
In summary, **this code sanitizes HTML from Microsoft Forms form content while allowing the text to be styled**.
For instance, you create a form at Microsoft Forms and style your text. This text is sent as pure HTML to the server, and when you see the form, this HTML is filtered using this function.
**You must bypass this function to render the whole string as HTML**. Otherwise, the string will be rendered as text. 
So, can you find the stored XSS?

 **Let's start with bug #1!**

## variable `e` only match valid, closing html tags
The variable match valid, **closing** HTML tags, while browsers will tolerate broken HTML. 
So, consider the string :
`<u><span>w<3<br><br</span>id<span><ol><img src=x onerror=alert(1)<br><em><span><u><ul><ol><span><i>` 

The `<img>` is never closed. So it bypasses the check of variable e, and the result (without the junk tags) will be `<img src=x onerror="alert(1)<br">` The javascript is broken, but it's still working, and you can fix it easily with a different payload. That's the first bug I discovered. 

The vulnerability was discovered and reported on **9.10.2022** and was fixed by **21.11.2022**.

Thanks to Microsoft for the generous bounty :)

Microsoft fixed it by changing variable `e` to `var e=/(<|>)/i` Now they match every `<` or `>` without looking for valid HTML tags.* 


## variable `o` match anything inside parentheses

*Take a look at the variable o that matches the* ***color*** property! 
`o = /color:\s(rgb)\(([^)]+)\)(;)/gi` 

*this regex will match `color: rgb(0,0,0)`; and also a `color: rgb(non_valid,non_valid,non_valid);`

 ***This regex match anything from the open parentheses until they are closed**. Nothing prevents us from doing this: ```color: rgb("<img src=x onerror=alert`1`>//--222, 106, 25); ``` 
  by using `"`, we are first breaking the style attribute, then we inject our malicious HTML object/script, making sure to comment out the rest!
  
  Wait, but you can't use any `)` as this will close the rgb! So how can we execute Javascript like this?
We can use the **es6 template literals**, so```alert`1` ```  is just like `alert(1)`.
Of course, this POC can be optimized even more to prevent any leftovers of broken HTML tags on the page and to ensure execution even without es6.**

The vulnerability was discovered and reported on **9.12.2022** and was fixed by **28.12.2022**.
*Thanks to Microsoft for the generous bounty :)
Microsoft fix: `o = /color:\s*(rgb)\(\s*([0-9]+\s*,\s*[0-9]+\s*,\s*[0-9]+\s*)\)(;)/gi`*

## variable `r` does not match the same quotes on the `style` attribute

**Take a look at variable r:**
 `r = /<(span|b|i|u)\s?((style=)('|")\s*('|"))>/gi` 
 Simple and innocent, not that easy to spot, just allowing the style tag, and that should be ok with the later validation.
 
But this is another bug. **This regex does not validate matching quotes but does validate ANY quotes**. Something like `style='font-size: 1.2em;"` is valid. 
That means we could bypass the regex and inject malicious attributes. For example:

    <span style='color: rgb(120, 100, 192);font-size: 1.3em;">'onmouseover=alert(1) onmousemove=alert(1)

The vulnerability was discovered and reported on **29.12.2022** and was fixed by **10.02.2023**.
Thanks to Microsoft for the generous bounty :)

The variable r was slightly different in the original POC (as a result of other bugs I found), But the concept is the same.


I give my honest respect to Microsoft again for responding and fixing those bugs asap, and for giving me the option to buy a car from their bounty money.

as result of all those vulnerabilities I discovered , I got into the MSRC Leaderboard of Q4 2022, at the 95th position. https://msrc.microsoft.com/leaderboard


## Author:
Guy Hayou

[Linkedin](https://www.linkedin.com/in/guy-h087/)

[HTB](https://app.hackthebox.com/profile/360735)
