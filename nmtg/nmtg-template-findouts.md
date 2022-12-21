Today I want to talk about the code, that we are doing for our LLazyEmail project right now.
To get more details, you can read my previous articles about this topic.

- About templates
- Why run a newsletter
- How I'm splitting a project into small pieces


As I want to improve our generator for email templates, we start to work on the third email template that we will include in our generator.


One of our goals - is code optimization, improvements, and coverage of different edge cases.

Сreatng a new template was an interesting experience. We mostly completed it. About 95% is done, I'm just lazy to complete it. Sliding into features building, instead of polishing HTML strings.

> This is why this project has value because code for emails is ugly and crushes easily.


So, while doing the third template and using outside help, I figured out things that can be improved in future iterations.



First: other people might have a different level of skills and knowledge in coding.

Always forget about it :)

Even a simple template get us into some trouble. So my assumption was not correct.

Result: I will invest more time and create detailed instructions with steps. Probably will publish an article here as well.

2) Errors reporting
Our template contains a lot of layers. Hierarchy is weird(still), so having validation and nice error reporting is crucial for debugging code and saving time.

3) We need to start to reuse our code.

I want to create a GitHub template with blank folders, files, configuration, etc.

4) I want to try some open-source modules that are doing the same thing. 
It sounds easier than it is. I tried to install a few npm modules, but our current setup with Rollup, Babel, and Jest get crashing for some strange reasons.
Result: I', trying to split our code into more pieces and attack this issue separately. will be a sandbox to play with new features. And later it will be a sub-module that will do just one thing and can be embedded into the current project.

5) And I want to organize Jest tests and commands that are calling them differently.
I already started exploring that way. Now, for each big piece that I have, I'm using a separate file and adding a number as a prefix. It helps me to keep a correct order in my "tests" folder.

And our package.json > scripts section starts to look ugly and unmanageable. I decided to check out how husky is organizing their scripts and moved some commands into bash scripts.

Automation!

The main goal is to spend less time converting the next template into pieces, compatible with our generator. And be able to migrate our logic into React Components in the future.

5) complex blocks require more attention, and a better evaluation before starting coding.

As our generator has some rules, template logic should obey to it. The current solution is not ideal. But it works. And as I noticed it, I created a task to try different options later.



Let me give you an example.

DB or API call returns a complex object that I need to display inside of our template.
How Should I pass it into our template?
Should I create several layers to display:
a section > a row > a block > an element?

or better just to create a chunk of that template, aka skeleton, and put everything inside of one file?

Plus, it's important to mention that sometimes people will fill in that data by hand. For example, creating content in Google Docs or Notion and exporting it. We need to handle those cases as well.


But I still need to get back to our bugs. If you are interested to hear more details -> share this article, give a comment, or just star our main repository on GitHub, so we get noticed by more developers.
