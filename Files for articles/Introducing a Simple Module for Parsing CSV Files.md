![image](https://user-images.githubusercontent.com/109110458/185234584-a5932690-24cf-4a89-9a8b-ade13844aa92.png)

Today I want to introduce a simple module for parsing CSV files

Recently I was exploring my old repository: https://github.com/Food-Static-Data/food-datasets-csv-parser

Inside I have a cool set of small modules, that helped me a lot. As my code is highly tied to those packages -> I need to pray for developers that build them, so I don't need to spend precious time.

List of modules that I'm using:
- csv-parser
- fs
- lodash
- path
- path-exists
- shelljs

Why did I create this package? It's simple. During our work @ Groceristar, we came around a number of databases and datasets, related to "food-tech". To be able to extract that data and play with it -> you need to parse CSV files.

![image](https://user-images.githubusercontent.com/109110458/185235031-ed9e4b21-182e-4481-8b36-90df8fba3718.png)

![image](https://user-images.githubusercontent.com/109110458/185235084-6cfc5270-ca9e-40c2-be81-1bb9abee2fef.png)

![image](https://user-images.githubusercontent.com/109110458/185235128-5f773f67-012c-4366-bcac-b2ca065e69dd.png)

![image](https://user-images.githubusercontent.com/109110458/185235168-83189e17-cbc3-4b19-80a1-d618f3dde58f.png)

Link to the repository: https://github.com/Food-Static-Data/food-datasets-csv-parser

Link to npm page: https://www.npmjs.com/package/@groceristar/food-dataset-csv-parser

I will also post updates about building modules for static data on indie hackers. While it didn't help with promotions a lot, founders are pretty positive people and their support really matters. Here is an org that I created few years ago: https://www.indiehackers.com/product/food-static-data

As usually, experienced developers might tell me that I'm stupid and CSV parsing is a mundane procedure. But I don't care. I realized that for a few separate projects we are running similar code. So I decided to isolate it.

I did it a few times before I finally find a way to make it work as I like. And you can see how it looks right now.

I can say, not ideal, but it was working fine for me. Right now I plan to revamp this package a little bit, in order to make it work with the latest versions of rollupjs and babel.

![image](https://user-images.githubusercontent.com/109110458/185235327-8b0e2cd1-3359-4da6-913f-cb827b91c5f4.png)

![image](https://user-images.githubusercontent.com/109110458/185235384-ae84e446-8750-4741-9b43-d85bbe1755f3.png)

![image](https://user-images.githubusercontent.com/109110458/185235442-af98c314-cc8d-4978-9bd8-d6b4913f667b.png)

![image](https://user-images.githubusercontent.com/109110458/185235486-c6405d45-fe2b-4521-b178-804e3988acf5.png)

![image](https://user-images.githubusercontent.com/109110458/185235534-06410b11-fc09-4dc5-8f8d-16870eada491.png)

![image](https://user-images.githubusercontent.com/109110458/185235590-c3bd6fad-4f54-49bd-abe0-73687a903598.png)

While the idea is simple: connecting a dataset in CSV format, parsing it, and exporting data as you need it, while you need to make it work with 10 different datasets, things arent as easy as they sound in your head.

CSVs not only related to food tech datasets. But for me was important to be able to use different datasets and easy to play with it. It makes other modules that we are building data-agnostic and more independent to a database/frameworks/logic. Basically, around this idea, we created and optimized like 13 repositories. Recently I created a separate organization that will be focused on those repositories only.

Link: https://github.com/Food-Static-Data

Later I plan to remove some repositories when they wouldn't be replaced by other, more popular, and stable tools. This current module can be useful for parsing other datasets too. But making it separate from the food tech topic isn't my task at this point.

And I was able to include and implement cool and important packages, like husky and coveralls. I can't say that I get most from them, but at the same time, it helped me to jump into the "open source ocean" that related to the GitHub rabbit hole that I'm still exploring for so many years.

and it's good to not just type another line of code, but also be able to see that your codebase is solid and nothing breaking it behind your back

![image](https://user-images.githubusercontent.com/109110458/185235786-d62fb64b-de4d-421a-9da0-b09e009c2826.png)

![image](https://user-images.githubusercontent.com/109110458/185235825-33336398-411c-498d-a081-cb2293255bd8.png)

CodeClimate(https://codeclimate.com/) helped me to explore and be able to take another look at how to develop things.

![image](https://user-images.githubusercontent.com/109110458/185235914-d3fa36ec-efbe-4e67-b04d-d1a5bbfa2da5.png)

![image](https://user-images.githubusercontent.com/109110458/185235992-aa056d4b-2647-468e-ac2f-50b6f1d8242d.png)

![image](https://user-images.githubusercontent.com/109110458/185236048-420b1600-a8b5-4be6-8eae-8be903f3a7b9.png)

Yeah, codeclimate shows that I have code duplicates and ~50h of tech debt. Looks like a lot, right? But this is a small independent package.
Imagine how much tech debt has your huge monolith project. Years of wasted time of developers, probably 🙂

At some point i'll remove duplicates and it will reduce number of hours on this page.

Plus, usually, your product owner or CTO is busy and can't review code and be able to track things inside your code.
CodeClimate can do some stuff for you. Just check those settings. Plus, they support open-source movement. So if your code is open and located on GitHub, you can use it for free.

![image](https://user-images.githubusercontent.com/109110458/185236163-65ceced9-81f3-4894-bf2e-fb982f309d11.png)

Stretch goals are simple
- I want to invest some time into CI/CD stuff. At this moment i'll pick Travic CI. At some point i'll extend it, so when a new version of this package is published, i'll run it against our datasets and will see if something breaks or not.
- I also need to remove duplicated code that i was moved into separated packages but still present here, due to back capability.
- and it's also not cool to see JS code with all there csv files at the same repository. I need to came with idea about how to organize folders and make it easy to navigate. While it works for me - other people might find it very confusing.

We even did a great readme file with an explanation of how to run this package

![image](https://user-images.githubusercontent.com/109110458/185236408-3c4d8cb1-3450-41ae-8ca1-9c7ed47f6e10.png)

![image](https://user-images.githubusercontent.com/109110458/185236450-75aab47c-1819-4991-aefb-a944eecf6a0b.png)

![image](https://user-images.githubusercontent.com/109110458/185236495-72f44138-1814-415d-b3e7-bc5dee1c75f5.png)

![image](https://user-images.githubusercontent.com/109110458/185236535-f0bb4b3a-88fb-435a-b088-9e87604c7368.png)

We collected a great number of datasets that can help a wast number of food projects. Some of them even sell the latest updates for money.
Right now this module was tested with:
- FoodComposition dataset
- USDA dataset(i pick 4 major tables with data)
- FAO(Food and Agriculture Organization of the United Nations) dataset
This module is not just for parsing data, we also have a need to write files in JSON format with formatted data inside.

Show some love if you want more articles like this one! any activity will be appreciated.

Similar articles:
- https://hackernoon.com/introducing-a-simple-npm-module-with-email-templates
- https://hackernoon.com/how-i-started-to-build-react-components-for-email-templates
- https://hackernoon.com/organizing-an-advanced-structure-for-html-email-template
- https://hackernoon.com/open-sourcing-regular-expressions-for-markdown-syntax-module
