Today I want to talk about code, that we are doing for our LLazyEmail project.
To get more details, you can read my previous articles about this topic.

- About templates
- Why to run a newsletter
- How i'm splitting a project into small pieces


As I want to improve our generator for email templates, we start to work on third email template thtat we will include into our generator.


One of our goals - code optimization, improvements and coverage of different edge cases.

Сreatng a new template was an interesting experience. We basically completed it. About 95% is done, im just lazy to complete it. Sliding into features building, instead of polishing HTML strings.

> This is why this project have a value, because code for emails is ugly and crushing easily.


So, while doing the third template and using an outside help, I figured out things that can be improved in the future itterations.



First: other people might have a different level of skills and knowledge in coding.

Always forget about it :)

Even a simple template get us into some trouble. So my assumption was not correct.

Result: I will invest more time and create a detailed instruction with steps. Probably will publish an article here as well.

2) Errors reporting
Our template contain a lot of layers. Hierarhy is weird(still), so having a validation and nice error reporting is crucial for debuggin code and saving time.

3) We need to start to reuse our code.

I want to create a github template with blank folders, files, configuration, etc.

4) I want to try some open-source modules that is doing the same thing. 
It sounds easier than it is. I tried to install a few npm modules, but our current setup with Rollup, Babel and Jest get crashing for some strange reasons.
Result: I', trying to split our code into more pieces and attack this issue separately. will be a sandbox to play with new features. And later it will be a sub-module that will do just one thing and can be embeded into current project.

5) And I want to organize Jest tests and commands that are calling them differently.
I already started exploring that way. Now, for each big piece that I have, i'm using a separate file and adding a number as a prefix. It helping me to keep a correct order in my "tests" folder.

And our package.json > scripts section start to look ugly and unmanagable. I decided to check out how husky is organizing their scripts and moved some commands into bash scripts.

Automation!

Main goal to spend less time to convert next template into pieces, compatible with our generator. And be able to migrate our logic into React Components in some point in the future.

5) complex blocks requiers more attention, a better evaluation before starting coding.

As our generator has some rules, template logic should obey to it. Current solution is not ideal. But it works. And as I noticed it, i created a task to try differrent options later.



Let me give you an example.

DB or API call returns a complex object that I need to display inside of our template.
How Should I pass it into our template?
Should I create number of X layer in order to display:
a section > a row > a block > an element?

or better just to create a chunk of that template, aka skeleton and put everything inside of one file?

Plus, it's important to mention that sometimes people will fill that data by hands. For example creating a content in Google Doc or Notion and export it. We need to handle those cases as well.


But I still need to get back to our bugs. If you are interested to hear more details -> share this article, give a comment or just star our main repository on GitHub, so we get noticed by more developers.
