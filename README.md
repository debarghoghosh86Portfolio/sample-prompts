# How I use AI to review docs before peer review

**Debargho Ghosh** · Senior Technical Writer · Bengaluru, India · [LinkedIn](https://www.linkedin.com/in/debargho-ghosh-8566ab23/)

> A quick note first: prompts 1 and 3 are cleaned-up versions of prompts I use at Ciena. The real ones point to internal style guides and products I can't share, so I've rewritten them around a made-up product called Acme Directory. Prompt 2 is a link check I put together for docs-as-code repos. None of the example text is real product documentation.

## Why I started doing this

Our peer reviews kept getting stuck on things that had nothing to do with technical accuracy: a term spelled three different ways, a prerequisite buried at the end, steps in the wrong order. So I started running drafts through a few prompts before they went to a reviewer. It's a simple idea, but it frees reviewers up for the questions only an engineer or a senior writer can answer.

At Ciena, 8 writers now use these prompts across roughly 10 guides, and they've shortened our review cycles.

## How it fits into my day

1. I finish the draft in AsciiDoc.
2. I run the three prompts below in an AI tool the company has approved.
3. I go through the flags one by one and decide what to fix. The AI never touches the source file.
4. Anything technical gets checked against the spec, or I ask an engineer.
5. Then the draft goes to peer review.

## Prompt 1: Structure

This one catches procedures that would trip an admin up halfway through.

```text
You are reviewing a task-based procedure written for system administrators.
Check the draft below against these rules and list every problem you find.
Do not rewrite the document.

Rules:
1. The topic opens with one sentence saying what the task achieves.
2. Prerequisites (roles, versions, required settings) appear before the first step.
3. Each step contains one action, starts with a verb, and appears in the order
   the user performs it.
4. Results are stated after the step that produces them.
5. The procedure ends with a way to verify the task worked.

For each problem, give the rule number, the step or line, and a one-line
explanation. If you are not sure something is a problem, say so instead of
guessing.

Draft:
<paste draft>
```

## Prompt 2: Broken links and orphan topics

In a docs-as-code repo, links break quietly, usually when someone renames a file or a heading. New pages also get forgotten in the navigation. This prompt compares the draft with the real file list and nav file, so it can't just guess.

```text
You are checking links in a docs-as-code repository. I will give you three
inputs:
1. The draft topic (AsciiDoc)
2. Every file path in the docs folder, with the section IDs and xref links
   in each file
3. The navigation file (nav.adoc)

Check the draft and report:
- Broken internal links: an xref to a file that is not in the file list, or
  to a section ID (#id) that does not exist in that file
- Orphan topic: the draft is missing from the navigation file AND no other
  file links to it
- Unhelpful link text, such as "click here" or "this link"
- External URLs, listed separately so I can test them with a link checker

Return a table with three columns: link or topic | problem | suggested fix.
Base every finding only on the inputs provided. If a file or anchor is not
in the list, treat it as missing; do not assume it exists.

Draft:
<paste draft>

File list:
<paste file list>

Navigation file:
<paste nav.adoc>
```

## Prompt 3: Terminology

When several writers share a doc set, terms drift. One person writes "user group", another writes "group of users", and readers start wondering if they're different things.

```text
Compare the draft below with the approved terminology list.
Flag every term that does not match the list, including differences in
spelling and capitalization, and any concept named in more than one way
(for example, "user group" and "group of users").

Return a table with three columns: term in draft | approved term | location.
Use only the terminology list provided. Do not suggest new terms.

Terminology list:
<paste list>

Draft:
<paste draft>
```

## A before-and-after example

### The first draft (`configure-sso.adoc`, not yet in `nav.adoc`)

```asciidoc
= Configuring SSO

Go to the settings and enable SAML. It should be configured with the IdP
metadata, see xref:idp-setup.adoc#metadata[here]. Click Save. Note that you
need admin rights to do this. Users will now be able to log in using SSO.
```

### What the prompts flagged

| Check | Flag |
| --- | --- |
| Structure | No opening sentence saying what the task achieves (rule 1) |
| Structure | The prerequisite (admin rights) comes after the steps (rule 2) |
| Structure | No way to verify the task worked (rule 5) |
| Links | `idp-setup.adoc` is not in the file list; the closest match is `idp-configuration.adoc` |
| Links | Section ID `#metadata` doesn't exist; the matching section in `idp-configuration.adoc` has the ID `export-idp-metadata` |
| Links | Link text "here" doesn't say where the link goes |
| Links | Orphan topic: `configure-sso.adoc` isn't in `nav.adoc` and no other page links to it |
| Terminology | "log in using SSO" should be "sign in with single sign-on (SSO)" |
| Terminology | "admin rights" should be "Administrator role" |

### After my revision

```asciidoc
= Configure single sign-on (SSO)

Set up SAML-based SSO so that users can sign in to Acme Directory with your
identity provider (IdP).

.Prerequisites
* You have the Administrator role.
* You have your IdP's metadata XML file. To get it, see
  xref:idp-configuration.adoc#export-idp-metadata[Export your IdP metadata].

.Procedure
. In the admin console, go to *Settings > Authentication*.
. Turn on *SAML SSO*.
. Upload your IdP metadata XML file.
. Click *Save*.

.Verification
* Sign out, then select *Sign in with SSO* on the sign-in page.
  You are redirected to your IdP.
```

And one line added to `nav.adoc` under Authentication, so the page is no longer an orphan:

```asciidoc
** xref:configure-sso.adoc[]
```

## How I keep the AI honest

The prompts only flag problems. They don't rewrite anything, so every change is still my call. I also tell the model to say when it isn't sure, because a confident wrong answer costs more time than an honest question.

I never take technical details from AI output. Parameter names, commands, UI labels and version numbers get checked against the spec, the product itself, or an engineer. The same goes for links: the link prompt is good at spotting missing files and anchors, but it can't tell whether a URL is live, so external links go through an automated link checker.

Internal content only goes into AI tools the company has approved. And at Ciena, I also helped build a prototype RAG tool that checks our docs against engineering specs, so the checks rest on source material rather than on the model's general knowledge.
