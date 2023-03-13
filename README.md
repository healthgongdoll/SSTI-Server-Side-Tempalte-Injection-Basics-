# SSTI-Server-Side-Tempalte-Injection-Basics-

## Introduction

This is Jae Young's Web Security Study Note for SSTI (Server Side Template Injection).</br>
You will see a lot of SSTI problems in CTF, Web vulnearbilities. </br>
To understand the concept, I made this study guide.</br>

## What is template?

![image](https://user-images.githubusercontent.com/79100627/224593402-2880fdfa-0073-4d6f-909d-1935978ff1d6.png)

Templates are essentially files that contain a bunch of static content and within that static content, we have a spots where dynamic content is supposed to go.</br>
We denote where that dynamic content goes with specific templating syntax that's understood by the templating engine in use.

RUBY -> ERB Syntax -> <%= @user.name %>
Python -> Jinja syntax -> {{user.hit_that_like}}

within ths symbol {{ }} or <%= %> , you will see a reference to something like a model or an object and its attributes. <br/>
So if we call user.username, the templating engine will see that and it will dynamically put the user's username. <br/>

If we are putting voting_location.description, then we will see the description gets populated there. 

Understanding SSTI, that the root of the issue is that our payload is somehow rendered by the templating engine. (Modifying the templating file directly or parameter happens to be sent and rendered by the templating engine. (IT IS THE ROOT cause of this issue) 

## Examples of Template 
![image](https://user-images.githubusercontent.com/79100627/224594543-d2eeaefa-518b-4b04-99f1-de09228d2766.png)

you could have a user fill out data about their event, and template will acutally populate this data that's specific to the event that the user creates<br/>
So an application developer might create a template for events and then from there that user controlled data for each event will populated in the template, when a user goes to view that specific event 

![image](https://user-images.githubusercontent.com/79100627/224594619-7eb29eed-0c2a-406d-ad27-652b8692831b.png)
![image](https://user-images.githubusercontent.com/79100627/224595204-1dafbc8b-5e00-4fff-8137-b8c12f0dc94c.png)

Another tempaltes are email. Email templates are really common especially in modern frameworks where you create a static email template and then allocate specific parts that are dynamic so like the user's username or a specific link that the user will click.

![image](https://user-images.githubusercontent.com/79100627/224595468-cda14fee-c52f-4d6e-89d3-3095399b68d0.png)

## What about Server Side Template Injection ?

![image](https://user-images.githubusercontent.com/79100627/224595903-0f7e2fe8-87d3-4e17-9871-1f5356e78998.png)

In this example, here you can see an event with an event name and that event name is being passed dynamically as data in this case looks like some proper usage but what happens when user input is directly concatenated to the template and at that point it becomes part of the template directive and the template engine actually looks at that and does something with it or processes it based off what the user injects that's when we have an issue.

![image](https://user-images.githubusercontent.com/79100627/224596303-b3bc837f-28a4-41ec-a4e2-b601f09fbffa.png)
![image](https://user-images.githubusercontent.com/79100627/224596669-6d3b910b-5492-48ff-b958-1679c951d8c9.png)

Anywhere we look and we see user controlled input that's being reflected we want to see if when we inject that input does the templating directive look at it as data or does it actually interpret it and return the contents of what we try to evaluate this is where the power and the impact of ssti really comes into play. because if we can confirm that the templing engine evaluates simple arithmetic like seven times seven now we could try something like system("whoami") and possibly get rce and even if we can't get rce we can use the power of the templating engine to interact with functionality we shouldn't be albe to interact with and possibly exfiltrate some sensitive data

![image](https://user-images.githubusercontent.com/79100627/224597156-f1c6bb4b-ee21-4efc-bcd8-49bc346bf365.png)

## SSTI Discovery Method 

1. Look for reflection of our user-controlled input 
 
we need a reflection of input that way we know when our payload is processed by the templating engine do we actually see that its evaluated or is it returned as such without evaluation 

2. If our payload is evaluted, enumerate the templating engine 

we have a list of payloads that if they get evaluated by a templating engine would return 7x7 = 49 

![image](https://user-images.githubusercontent.com/79100627/224598999-cd01a28d-66c7-450b-b23c-4a82e92f3d99.png)

3. Exploitation! 

If the sample payloads on internet does not work, you need to find a way by going through documentation or exploring object like self and its available methods or different functions that you have disposal. Error message do make this easier but if all you get is ouput of properly evaluated templating syntax, then you need to get crafty. 

## Example

![image](https://user-images.githubusercontent.com/79100627/224599600-5ca83601-f649-4495-8635-7e8de5887192.png)

Let's say there is a shopping website and when we click teh Adult space hopper, Unfortunately this product is out of stock message pops up. In Burp Suite /?message=Unfortunately%20 ... if we take this request to repeater with ctrl r (Or html), we can see where this message comes from 

![image](https://user-images.githubusercontent.com/79100627/224599809-acb9a537-ddba-4cc5-8598-aec21c056f2d.png)

![image](https://user-images.githubusercontent.com/79100627/224599988-41bf82a2-69a1-42f5-bb05-1935d7c3bfaf.png)

in this GET Method we can modify the /?message= field with list of payloads 

1. {{7*7}}
2. ${7*7}
3. <%= 7*7 %>
4. ${{7*7}}
5. #{7*7}

if the payloads being sent to a templating engine and actually being evaluated it should return 49 

