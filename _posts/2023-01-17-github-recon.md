---
title: "How Github Recon got me an Authenticated API Access of Target"
date: 2023-01-17
categories:
  - VAPT
tags:
  - Penetration Testing
  - Github Recon
  - Web App 
  - API
---

I just got off from a WebApp Pentesting engagement of one of the Client's asset.

The engagement is of type Black Box Testing with only scoped URLs provided.

The scope included one subdomain and associated API endpoints.
For the sake of this article and respecting the NDA, I will refer to it as:
`dashboard.target.com` and `api.target.com/v2`

Not much can be done in the Subdomain Reconnaissance phase as the scope is already defined with one subdomain.

I then, went after the GitHub Recon in the hopes of finding its repositories or some other useful information.

I tried several Search Filters and searched for keywords related to the `dashboard.target.com` and `api.target.com` but obviously all their repositories were Private returning no data.

I then broadened my search and looked for public repositories with the same name as of the client's, to which I got several Repos (around 10).
All these public repos were unfinished and unmanaged since a year and was built using Angular.
Having worked with it before I knew the Environment variables, APIs and Tokens are stored in `environment.*.ts`, `*.service.ts` and `*.interceptor.service.ts` files respectively.

Out of these repos, I found that 2 repos had hardcoded API domain and Token 
though the API domain is from **development** env `api.dev.target.com`

Following are the contents of those.

![[image-12.png]]


![[image-15.png]]


Hence, I got the following data:
- API dev domain
- Access Token
- API endpoints extracted from several `service.ts` files

The commits where from 2021 so I was expecting the Tokens would have expired by now.
Still giving it a shot, I hit the dev endpoint and BINGO! The API returned data with status code 200.

![[image-9.png]]


The same dev token didn't work for the prod `api.target.com` thereby inferring that the developers hardcoded those values for development but also unknowingly pushed the code in public repositories.

Since this Finding is from `api.dev.target.com`  which is out of scope, I responsibly reported it over the mail as it can't be included in a Pentesting Report.

They immediately terminated the token.

![[image-10.png]]

Rest in Pieces, the token :)