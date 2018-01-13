---
title: Django doc - Index
date: 2017-12-22 18:09:11
tags: [Django, Django-documentation]
---

Everything you need to know about Django.

<!-- more -->

## 文档的组织结构

Django 的文档有很多，文档的高水平组织方式会帮你了解去哪能够找到相应的内容：

+ [Tutorials](#) 会手把手教你一步步创建一个 web 应用，如果你在 Django 或者 Web 应用开发方面是个新手的话建议你从这开始，同样也可以访问下面的 ["First steps"](#)。

+ [Topic guides](#) 会在较高层次讨论一些关键词和概念，并且提供一些有用的背景信息和解释。

+ [Reference guides](#) 包含 API 的技术参考和 Django一些其他方面的知识。

+ [How-to guides](#) 会帮你通过关键字和用例来解决相关问题，使用这部分文档相比 __tutorials__ 需要更高水平的知识，并且希望你了解 Django是如何工作的。

## First steps

如果你是新手的话建议从这里开始。

+ 从0开始：
  + 预览
  + 安装

+ 教程：
  + Part 1: Requests and responses
  + Part 2: Models and the admin site
  + Part 3: Views and templates
  + Part 4: Forms and generic views
  + Part 5: Testing
  + Part 6: Static files
  + Part 7: Customizing the admin site

+ 进阶教程
  + 如何编写和复用的 apps
  + 写下 Django 的第一个 patch

## The model layer

Django 在 Web 应用中为组织和操作数据提供了一个抽象层（即 model 层），更多：

+ Models:
  + Introduction to models
  + Field types
  + Indexes
  + Meta options
  + Model class

+ QuerySets:
  + Making queries
  + QuerySet method reference
  + Lookup expressions

+ Model instances:
  + Instance methods
  + Accessing related objects

+ Migrations:
  + Introduction to Migrations
  + Operations reference
  + SchemaEditor
  + Writing migrations

+ Advanced:
  + Managers
  + Raw SQL
  + Transactions
  + Aggregation
  + Search
  + Custom fields
  + Multiple databases
  + Custom lookups
  + Query Expressions
  + Conditional Expressions
  + Database Functions

+ Other:
  + Supported databases
  + Legacy databases
  + Providing initial data
  + Optimize database access
  + PostgreSQL specific features

## The view layer

+ The basics:
  + URLconfs
  + View functions
  + Shortcuts
  + Decorators

+ Reference:
  + Built-in Views
  + Request/response objects
  + TemplateResponse objects

+ File uploads:
  + Overview
  + File objects
  + Storage API
  + Managing files
  + Custom storage

+ Class-based views:
  + Overview
  + Built-in display views
  + Built-in editing views
  + Using mixins
  + API reference
  + Flattened index

+ Advanced:
  + Generating CSV
  + Generating PDF

+ Middleware:
  + Overview
  + Built-in middleware classes

## The template layer

+ The basics: Overview

+ For designers:
  + Language overview
  + Built-in tags and filters
  + Humanization

+ For programmers:
  + Template API
  + Custom tags and filters