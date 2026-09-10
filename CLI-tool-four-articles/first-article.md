# The $0 Email Engine: How I Used Notion, GitHub, and Amazon SES to Outsmart $400/mo ESPs

## The Problem: Bloated ESPs and Monthly Enterprise Ransoms

If you walk into any modern marketing department or tech startup, you'll find them paying $300 to $1,000 every single month for bloated Email Service Providers (ESPs) like Mailchimp, SendGrid, or Klaviyo.

And what do they get for that monthly enterprise ransom? 
- A slow, laggy drag-and-drop editor
- A clunky template system
- A dashboard that locks up every time you try to format a simple bulleted list

As a developer, manager, and magazine editor, I looked at that whole ecosystem and thought: **This is complete garbage.**

I didn't want to spend $400 a month just to push text and HTML through an API pipe. More importantly, as a self-proclaimed procrastinator who struggles with the sheer friction of finishing massive tasks, I needed a workflow that *removed* friction instead of adding it.

So I built my own lean, hyper-modular "production engine" using three free developer-friendly tools: **Notion**, **GitHub/GitLab**, and **Amazon SES**.

Here is how I set up a zero-dollar infrastructure machine, why it completely changes the email workflow, and how it saved me from operational burnout.

---

## 1. Notion: The Frictionless Brain Dump Station

The biggest mistake people make with newsletters is trying to write, edit, format, and layout an email all inside their ESP's template builder. It's an absolute nightmare. You get distracted by button colors, worried about how the preview looks in Outlook, and end up spending hours on formatting instead of writing.

I started doing 100% of my ideation, rants, and drafting inside **Notion**.

### Why Notion?

- **Zero-Friction Editing**: It's clean, fast, and stays out of your way.
- **Markdown Support**: I can draft an entire technical piece, drop in code blocks, bold text, and images, and hit Export to Markdown.
- **Modular Thinking**: I can break down a giant 3,000-word newsletter campaign into tiny, bite-sized sub-tasks, cards, or toggle lists.

When inspiration hits, I don't open an email tool. I open a blank Notion doc, dump my raw thoughts, and get out. No UI getting in the way. No "save" buttons. Just pure content creation.

---

## 2. GitHub / GitLab: Staging, Version Control, and Free Hosting

Once a draft is exported to Markdown or converted into raw HTML, where does it go?

Most marketers copy-paste code into an ESP's draft folder, where it gets lost, overwritten, or broken by the WYSIWYG editor's auto-formatting algorithms.

Instead, I push my email assets straight to a public or private GitHub/GitLab repository.

### Repository Structure Example

```
my-email-pipeline/
├── issue-01/
│   ├── raw_draft.md
│   ├── index.html
│   └── assets/
│       ├── header.png
│       └── diagram.svg
├── issue-02/
│   ├── raw_draft.md
│   ├── index.html
│   └── assets/
└── archive/
    └── previous-campaigns/
```

### Using Git as the Middle Layer

Using Git as the middle layer of an email pipeline gives you superpowers:

- **Total Version Control**: If a CSS tweak breaks the layout in Outlook, I don't panic. I just run `git diff` or rollback to the previous commit.
- **Free CDN for Images**: Instead of uploading assets to a clunky ESP media library, I host email images directly via GitHub Pages or GitLab static hosting. They render fast, never disappear, and cost $0.
- **Sanitize Your Code**: You can inspect raw HTML, inline CSS tags, and links without some SaaS platform injecting unwanted tracking scripts or mangling your HTML structure.

---

## 3. Amazon SES: The $0.10 Infrastructure Muscle

Once the HTML/CSS code is stored safely on Git, the actual "sending" step is purely transactional.

Instead of routing those emails through an expensive marketing SaaS, I hook my system directly to **Amazon Simple Email Service (SES)**.

As we covered in our deep dive into SES, Amazon charges a flat rate of **$0.10 per 1,000 emails**.

### Cost Comparison: Mailchimp vs. Amazon SES

| Provider | 50,000 Emails/Month | Yearly Cost |
|----------|-------------------|------------|
| Mailchimp | ~$350.00 | ~$4,200 |
| Amazon SES | $5.00 | $60 |
| **Savings** | **$345/month** | **$4,140/year** |

You aren't paying a $345/month tax for a pretty dashboard you don't need. You are paying purely for raw bandwidth and delivery infrastructure backed by AWS.

---

## The Secret Weapon: Outsourcing and Modular Delegation

This stack wasn't just built to save money—it was engineered to accommodate my own personal workflow flaws.

I'm the first to admit my English isn't perfect. I didn't go to an English-speaking school, and while my raw style is distinct and sarcastic, my drafts can be littered with bad verb tenses and missing articles. 

Trying to fix all my grammar mistakes while wrestling with HTML code in SendGrid was a recipe for procrastination. I would literally abandon articles halfway through.

By decoupling my stack into **Notion + Git + SES**, I created a modular assembly line that made delegation effortless:

### The Four-Step Pipeline

1. **The Raw Dump**: I write my raw, unfiltered technical ideas in Notion.
2. **The Ghostwriter Hand-Off**: I hand that Notion doc or Markdown file off to a professional copywriter. She takes my raw technical outline, cleans up the grammar, smooths the flow, and keeps my exact "bad boy" voice intact.
3. **The Code Commit**: The final text is converted to inline HTML/CSS and pushed to GitHub.
4. **The Trigger**: The production email is dispatched directly via Amazon SES.

Because my sending infrastructure (SES) is completely separated from my content creation layer (Notion/Git), I can give freelancers access to docs or code repositories without ever giving them the keys to the sending system. Zero security risk. Clean separation of concerns.

---

## Why We Need to Productize the "Lazy Stack"

Building a pipeline out of Notion, Git, and Amazon SES feels like a massive victory for lean developers. You feel like you outsmarted the entire SaaS industry by building an enterprise-grade sending engine for pennies.

But let's be honest about the manual friction:

- Copy-pasting Markdown into inline HTML tools takes time.
- Managing image links manually in Git repositories can be tedious.
- Trimming inline CSS to avoid Gmail clipping requires manual sanity checks.
- Coordinating between three different tools creates context-switching overhead.

It's an incredible hack, but eventually, you hit a ceiling where the manual glue holding these tools together starts to wear you out.

And that exact friction point—taking this dirt-cheap, ultra-lean, modular developer workflow and wrapping it in a seamless automation layer—is the entire reason I started building **Lazy Email**.

---

## The Bottom Line

If you are tired of paying enterprise prices for basic email tools, **stop using monolithic marketing platforms**. 

Split your tasks. Embrace free developer tools. Let raw AWS infrastructure do the heavy lifting. Build your own email engine, own your data, and reclaim $345 every single month.

The future of email marketing isn't a SaaS subscription. It's a modular, developer-friendly pipeline that you control.
