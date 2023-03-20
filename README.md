# Advocating-for-2FA
![image](https://user-images.githubusercontent.com/37233306/226416436-f4af1a4d-def7-4392-82d2-cb7346007fd4.png)

----------------------------------------------------------

## [1] Analyze the target : 
we will start from this url https://oup0leb6eit.hackday.fr, it will show coming soon page
if we go to robots.txt file , it will show the path of the sitemap as shown in the photo :

![Screenshot from 2023-03-19 22-09-42](https://user-images.githubusercontent.com/37233306/226417650-ac667855-c424-4cba-a952-988147636696.png)
so we will visit the sitemap.xml , note that the path was http://localhost:8080/sitemap.xml 
don't worry we will not perform any kind of ssrf or lfi here , sitemap.txt will open directly
![Screenshot from 2023-03-19 22-09-58](https://user-images.githubusercontent.com/37233306/226417983-51d7b8f9-e5ec-4163-ba49-d7246947764e.png)
as shown in the photo , the sitemap.txt file contain two pages rather that the home , login and register 
we will go to the registration page to register a new user
![Screenshot from 2023-03-19 22-09-29](https://user-images.githubusercontent.com/37233306/226418539-f97c04e8-ffe9-4701-9891-b43d77e72418.png)
as shown in the photo and like any registration panel , we have to provide a username  , password and confirm that password
I will use github1 for the username and the password as shown below 
![Screenshot from 2023-03-19 22-10-30](https://user-images.githubusercontent.com/37233306/226419963-040ff2a6-35a0-43af-be1e-a91b014786b6.png)
everything is ok untill now 

## [2] 2FA and it's bypass : 
so when we confirm our login it will require a 2FA code , but how we can get the code ? 
as shown below the challenge provide you a way to generate new 2FA instance 
![Screenshot from 2023-03-19 22-10-48](https://user-images.githubusercontent.com/37233306/226420615-4b969910-b224-4d72-bcc7-9f5eb103cb4b.png)
everything is beautiful , let's create the instance and complete 
when we create the instance it will show a QR code as the photo below
![Screenshot from 2023-03-19 22-10-53](https://user-images.githubusercontent.com/37233306/226420945-c02c77d9-1a65-4443-b47a-4983c3589771.png)
note that there many famous applications to use it for this kind of authentication , I will use google authenticator 
but why not to see what the QR contains , so let's use a decoder 
as shown below when I decode it online
![Screenshot from 2023-03-19 22-11-39](https://user-images.githubusercontent.com/37233306/226421468-07624b6b-08fe-461e-b732-d6ade992a4b1.png)
we can notice this string **otpauth://totp/Web%20Challenge%20-%20Authentification%20%28github1%29?secret=GVSDINBYMQZWKOLDHFTGIMJWMNQWKMTEHBSTGNBWGVRDOOJRGBSA&issuer=Hackday**
so we can see **github1** user clearly , but what is that secret ??
if we put it's value on https://gchq.github.io/CyberChef we will notice that the value is base32 decoded 

![Screenshot from 2023-03-20 20-40-15](https://user-images.githubusercontent.com/37233306/226422799-1b401cc7-9a98-4e45-8e0a-a65f15afd045.png)

but what is the output ??
the first impression was MD5 , so I have tried to crack it , but no results , so what ? stop ? of course no 
then I have tried to do hash the username as MD5

![image](https://user-images.githubusercontent.com/37233306/226423520-7f7318a3-54ea-4b84-a881-7d3ecf7001dc.png)

so why not to do the MD5 again ? 

![image](https://user-images.githubusercontent.com/37233306/226423661-5348bcac-55df-4ada-b2ef-e5256ce5ca77.png)

BOOOOOOM!!!!!!
so that is the secret : BASE32(MD5(MD5(USERNAME)))
but wait a minute , what username would we choose ? 
admin , Admin , ADMIN , besthacker ???
let's take a step to the back , if we will go to the main page we will found this JavaScript file https://oup0ieb6eit.hackday.fr/js/app.c323a17d.js 
I have captured the most important part of it 

```javascript
                function Ce(e, a, t, s, o, i) {
                    return (0, r.wg)(), (0, r.iD)("div", ye, [(0, r._)("div", Ae, [o.message ? ((0, r.wg)(), (0, r.iD)("div", Fe, (0, v.zw)(o.message), 1)) : (0, r.kq)("", !0), (0, r._)("h1", null, "Welcome " + (0, v.zw)(o.username), 1), (0, r._)("button", {
                        id: "logout-btn",
                        onClick: a[0] || (a[0] = (0, n.iM)(((...e) => i.logOut && i.logOut(...e)), ["prevent"]))
                    }, "Logout"), "admin" === o.username ? ((0, r.wg)(), (0, r.iD)("h3", Le, [(0, r.Uk)(" Congratulations !!! You found the flag ;) ... "), (0, r._)("img", {
                        src: o.flag,
                        alt: "IMG error",
                        style: {
                            display: "none"
                        }
                    }, null, 8, Se)])) : (0, r.kq)("", !0)]), Ue])
                }
 ```
so now we know that we must use **admin** username , but what is the e.token ? 
Do't worry we will deal with it later 
returning to the QR code it self we can do this equation on **admin** username BASE32(MD5(MD5(admin))) then put as secret value

so it will be like this  
**otpauth://totp/Web%20Challenge%20-%20Authentification%20%28admin%29?secret=MMZTEOBUMQYGMOJUGYYDMZDFGFTGIMTBMYYTOMTBMJQTCNLCMYZQ=&issuer=Hackday**
after that we can use any online tool to generate a QR code from text .
once we generate the QR code we can scan it using google authenticator
So now we know how to get the admin OTP 

## [3] JWT part 

you can see that there is a header called **X-Access-Token** and it's value is JWT token generated for the user who logged in 

![image](https://user-images.githubusercontent.com/37233306/226442955-1e7c824e-6950-4e20-8550-82bbbe1f51c1.png)

if we copy it and try to modify it using jwt.io 

![image](https://user-images.githubusercontent.com/37233306/226443408-7ac385e3-94f2-4b9e-a5d2-1f34483aebe2.png)

we cannot just edit it and replace it by the new one , we need to find the secret of the JWT 
but how ? to understand how jwt works and what is the vulnerabilities associated with you can check this link https://portswigger.net/web-security/jwt
![Screenshot from 2023-03-19 22-16-59](https://user-images.githubusercontent.com/37233306/226445321-6dc92bca-b3cb-43a4-8317-52dc70a4e077.png)

I will crack it using hashcat 

**hashcat jwt.txt -m 16500 ** 

![image](https://user-images.githubusercontent.com/37233306/226444430-b3f52f7c-86b3-470f-b74f-fc4afdaca4f2.png)

then ???? 
let's return to 2FA , we will put the token of the user we want and his otp 


![Screenshot from 2023-03-19 22-16-46](https://user-images.githubusercontent.com/37233306/226444809-b9b108c0-52b7-4bd4-9279-18b50a5413a5.png)

then we will see flag page : 

![Screenshot from 2023-03-19 22-16-52](https://user-images.githubusercontent.com/37233306/226444959-448d7e23-73fa-4b05-a829-3452e672a7bb.png)

the flag page will connect /getinfo endpoint in this api https://ahjiechae6r.hackday.fr
if the token was the admin token , then the flag page will return welcome admin and congrats , as shown 


![Screenshot from 2023-03-19 22-16-59](https://user-images.githubusercontent.com/37233306/226445308-1d8fdcc1-8973-41f8-88fd-1770c488bb8b.png)

then it will redirect you to strange endpoint , if you decode it you will get the flag  

![Screenshot from 2023-03-19 22-17-20](https://user-images.githubusercontent.com/37233306/226445447-b47a27a6-62ae-4a51-84de-52fb552aae2c.png)



Thanks for reading my writeup , it was amazing question , but everybody was bruteforcing :) 


