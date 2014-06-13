---
layout: post
title: "MS Unit Testing"
description: "A short list of attributes for Microsoft Unit Testing Framework to get started"
category: "notes-tips-tricks"
tags: ["TDD"]
---
{% include JB/setup %}

A short list of attributes for Microsoft Unit Testing Framework to get started:

**[TestClass]** - you should use this attribute for each class, which has methods with the attribute TestMethod. If there is inheritance, it would be better to use this attribute for all base and derived classes which have attributes related to this test framework.

**[TestMethod]** - this attribute is used to mark test methods.

**[TestInitialize]** - before each test method MS Unit Test Framework will fullfil the code of a method with this attribute.

**[TestCleanup]** - after each test was be completed method with this attribute will be executed.

**[AssemblyInitialize]** - a method will be executed one time before all test methods.

**[AssemblyCleanup]** - a method will be executed one time after all tests methods.

Methods with last 4 attributes must have only one argument of TestContext.

**NB**: if you forget to use **[TestClass]** attribute for the class which has test methods with appropriate attributes, you will be notified about it. But methods with **[AssemblyInitialize]** attribute will be ignored if the class does not have TestClass attribute.