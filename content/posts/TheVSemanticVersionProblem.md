---
​---
title: "The Irresistible Urge to Add a \"v\" to Semantic Versions – And Why to Avoid It"
description: "In past releases from well-known and widely-used companies, problems (on a global scale) have occurred due to the use of “v” in semantic versions."
summary: "In past releases from well-known and widely-used companies, problems (on a global scale) have occurred due to the use of “v” in semantic versions."
date: 2025-05-08
tags: [git, semantic version]
categories: ["git"]
series: ["git"]
author: ["Pieroci"]
ShowToc: false
TocOpen: false
draft: true
weight: 5
cover:
  image: "../vSemanticVers.png" # image path/url
  alt: "The Irresistible Urge to Add a \"v\" to Semantic Versions – And Why to Avoid It"
  caption: "The Irresistible Urge to Add a \"v\" to Semantic Versions – And Why to Avoid It"
  relative: true # when using page bundles set this to true
  hidden: true # only hide on current single page
editPost:
  URL: "https://github.com/pieroci/content"
  Text: "Suggest Changes" # edit text
  appendFilePath: true # to append file path to Edit link
​---
---

# The Irresistible Urge to Add a "v" to Semantic Versions – And Why to Avoid It

This article was written to educate and encourage (but also to warn), and aims to get you thinking about the "v" prefix in semantic versions used to represent software versioning.

If you use the semantic versioning standard in your code to distinguish new software releases, you should be careful with the “v” character…

## Case Study

In past releases from well-known and widely-used companies, problems (on a global scale) have occurred due to the use of “v” in semantic versions. Here are a few examples where this issue caused disruptions for minutes or even hours:

- IBM - Terraform: https://github.com/hashicorp/terraform-provider-local/issues/408
- Microsoft – Function Core Tool: https://github.com/Azure/azure-functions-core-tools/issues/4156

(I would have added more, but it required further research… Feel free to open an issue with similar cases and I’ll add them to the list.) 😊

## Why the Issue Happens

The problem stems from human error and the fact that even though the IT world strives for process standardization, standards aren't always followed. When releasing software, you don’t need the “v”. The `major.minor.patch` format is enough to identify a version.

“1.2.3” vs. “v1.2.3” = Release Problem!

### What Does the “v” Add?

Absolutely nothing. 😊

## What the Standard Says

The semantic versioning manifesto [https://semver.org/#is-v123-a-semantic-version](https://semver.org/#is-v123-a-semantic-version) explicitly states:

> **Is “v1.2.3” a semantic version?**
>
> No, “v1.2.3” is not a semantic version. However, prefixing a semantic version with a “v” is a common way (in English) to indicate it is a version number. Abbreviating “version” as “v” is often seen with version control. Example: `git tag v1.2.3 -m "Release version 1.2.3"`, in which case “v1.2.3” is a tag name and the semantic version is “1.2.3”.

## Reflection: How to Prevent the “v” Prefix in Semantic Versions

Here are some ways I imagined to prevent this kind of human error 😊

- **Remove the ability for users to create tags** for a specific commit on a Git server: Only automation should create tags.

- **Add a Git hook** to block tags prefixed with “v” that match the semantic version pattern.

### 🔧 Example: Server-side Pre-receive Hook

Save this in `.git/hooks/pre-receive` (with executable permissions):

```bash
#!/bin/bash

while read oldrev newrev refname; do
    if [[ "$refname" =~ ^refs/tags/ ]]; then
        tagname="${refname#refs/tags/}"
        if ! [[ "$tagname" =~ ^[0-9]+\.[0-9]+\.[0-9]+$ ]]; then
            echo "❌ Invalid tag '$tagname'. Use format X.Y.Z (e.g., 1.2.3)"
            exit 1
        fi
    fi
done

exit 0
```

### 📍 Client-side Pre-push Hook

In your local repo, create/edit `./git/hooks/pre-push`:

```bash
#!/bin/bash

while read local_ref local_sha remote_ref remote_sha; do
    if [[ "$local_ref" =~ ^refs/tags/ ]]; then
        tagname="${local_ref#refs/tags/}"
        if ! [[ "$tagname" =~ ^[0-9]+\.[0-9]+\.[0-9]+$ ]]; then
            echo "❌ Tag '$tagname' is not in the correct format X.Y.Z"
            exit 1
        fi
    fi
done

exit 0
```

### 🧪 Test Examples

| Tag          | Accepted? |
|--------------|-----------|
| `1.2.3`      | ✅        |
| `10.11.2000` | ✅        |
| `v1.2.3`     | ❌        |
| `1.2`        | ❌        |
| `1.2.3-beta` | ❌        |

## Promotion Flow and Tags

Manage your promotion flow exclusively through Pull Requests and apply tags automatically without the “v”. Ideally, a trunk-based approach with artifact/image promotion flow avoids environment-indicative suffixes in tags. Tags like “1.2.3-prd” or “1.2.3dev” should be avoided! If you’re using such patterns, you may want to redesign your promotion flow or branching strategy because you're losing the benefits of semantic versioning.

But who am I to judge? 😊 I won’t delve further into this now, but I’d like to explore the topic more in a future article.

As always, a fundamental part of every team and project: **Communicate!** The “v” is evil.

---

## Conclusion: Please, Don’t Use the “v”!

Does that “v” bring any benefits, or is it just useless repetition in your semantic version? I’ll let you decide 😊 I’ve already made up my mind.
