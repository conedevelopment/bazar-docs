---
posttype: "doc"
title: "Core Concepts"
description: "Definitions and core concepts."
icon: "/icons/core-concepts.svg"
github: "https://github.com/conedevelopment/bazar-docs/blob/main/core-concepts.md"
order: 0
---

## The Scope and Scale

Bazar is made to be compact and easy to integrate. This means our goal is to provide a simple solution for those who want to build a small to medium-sized e-commerce system. Keep it in mind when using Bazar; while it is relatively easy to scale (because of the Laravel foundations), it's not for the next eBay or Amazon.

## Headless

Bazar does not provide a REST API nor a ready-to-use web shop application. It behaves as a framework, and it gives every tool that helps you build your solutions.

If you need a REST API because you want to build a Vue-based store, you can easily do it. If you need a classic webshop application, no worries, you can make it relatively quickly and easily.

We'll provide example code and solutions in the near future, but the Bazar core will never contain more than necessary.

## Drop-in

Bazar is a drop-in solution; this means it behaves as a package and not as an application. It has its own layer, so you can build many different things in the same application next to an installed and running Bazar package.
