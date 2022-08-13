### setup email accounts with our domain name 
https://docs.hetzner.com/konsoleh/account-management/email/setting-up-an-email-account/

### server on Hetzner with Sendy(PHP script) and Amazone SES [Ivan]

An archive is located here: https://s3.console.aws.amazon.com/s3/object/arthur-courses-storage?region=us-east-1&prefix=sendy-5.2.zip

#### configure amazon SES with all stuff like dkim, sim, etc(but it will require a domain name probably)
#### Install a PHP script and try to send just one campaign
#### configure an ssl and email that we'll use from that domain for sending emails, like newsletters@domain.name

### create amazon ses account for email llazyemail@gmail.com


### he needs to think about how our code can be improved with DevOps practices. I'm sure for such simple code as we have - it will be done easily. 
probably we can start with GitHub Actions that we have in our lazymail* repositories, but it was never configured properly

### Configure GithubActions for all repos in LLazyEmail implement DevOps best practices [Ivan] 

### Fix workflow github

Айван мы все сломали...
давно правда
https://github.com/LLazyEmail/react-email-template/runs/6308254840

а файл тут https://github.com/LLazyEmail/react-email-template/blob/main/.github/workflows/workflows.yaml
