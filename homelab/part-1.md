# Part 1

*Date: 2025-02-25*

## Introduction

This blog post is the first in a series about setting up your own personal lab. The purpose is to document the necessary steps, which may also be helpful for beginners.

## Setting up the basics

### Baremetal

I am using the Beelink SER8 8845HS for this project. Since it is a barebone system, I purchased 2x32GB G.Skill DDR5 5600 and 2x2TB Samsung 990 PRO NVMe 4.0 x4 SSDs. The total cost is $1000.

[https://www.bee-link.com/products/beelink-ser8-8845hs](https://www.bee-link.com/products/beelink-ser8-8845hs)

[https://semiconductor.samsung.com/consumer-storage/internal-ssd/990-pro/](https://semiconductor.samsung.com/consumer-storage/internal-ssd/990-pro/)

[https://www.gskill.com/product/2/384/1683883295/F5-5600S4040A32GX2-RS](https://www.gskill.com/product/2/384/1683883295/F5-5600S4040A32GX2-RS)

### Redhat

The reason I chose Red Hat for my lab is that I need an operating system with strong support for virtualization and containerization. There are many options: Rocky, Alma, Ubuntu, etc., but I think I need to get familiar with Red Hat first since it is widely used at the enterprise level. In the future, I believe I will need to take a certification for it. Red Hat offers a developer program, so I can use it for free for one year, and after it expires, I can renew it. There are a few important links that you should save during this registration process:

[https://developers.redhat.com/register/](https://developers.redhat.com/register/)

[https://developers.redhat.com/products/rhel/download](https://developers.redhat.com/products/rhel/download)

[https://access.redhat.com/management/subscriptions](https://access.redhat.com/management/subscriptions)

[https://developers.redhat.com/articles/renew-your-red-hat-developer-program-subscription#frequently_asked_questions](https://developers.redhat.com/articles/renew-your-red-hat-developer-program-subscription#frequently_asked_questions)

## Lab diagram

![Diagram](.excalidraw "Diagram")
