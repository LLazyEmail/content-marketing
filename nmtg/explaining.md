### explaining new approach




So, we have your website, which is old and mature.(replace). It will be not easy to extend your project with new features, without touching your old wordress core. And an it was written on PHP - hiring will be challenging, because right now it's considered as an old fashion language.

Instead of opening your current server, editing files and adding something news, I propose to build small projects that will interract together.

I realized that when I opened you admin dahboard. While I still like that our team did wich custom types and everything else - my current experience tells me that it is better to simple things. And build a separate small tools-gears, instead of big.


An old way to fix or add something will be
- going to your code
- do some backups if something go bad
- building a new stuff
- make it work as it wouldnt break our current system
- do some code changes, upload changes on your server and enable those changes on your website.


Instead: a new feature or update will be done as a simple app that just connected to.....


[add add add]

SO imagine your house and for example you need more space now. Instead of ading an extrafloor and put whole system on stress, you just build a separate small house. Yeah, it has some drawbacks, but in software world it can give you some advantages. Plus I want do it that way.

New way, which i like. I know that it will take more time from begining for implementjing first features, it can work better and help to extend NMTG wit hnew thiungs.


[add section with proposal of milestones]


We will base our features on code that I build in different coding language. It is more popular and better tailored for curreent needs of Web.

And this code will be wrapped into small and simple multy-server functionality with limited logic.
Like each instance will be responsible for 1-5 functions max, not more.

It's called microservices architecture. Those servers will send/recieve some data, do callculaitons and it will communicate with oyur website via API. It can even have a different frontend under your domain name.

[extend it with cool schema, because it's not clear]

nmtg.com -> WP website

nmtg.com/%something% - a separate app that will receive a request and do simple things.

or we can do it 

app1.nmtg.com - another way for mapping sub-apps.
