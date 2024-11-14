---
description: introduction about the OS and documentation
---

# Introduction

This documentation covers the **Physical Memory Management** of the CertiKOS kernel as part of the CS422/522 lab assignment. CertiKOS is a certified operating system kernel built to provide foundational operating system features with a rigorous proof of functional correctness. This document focuses on three critical layers of physical memory management: **MATINTRO**, **MATINIT**, and **MATOP**.

#### Overview of Physical Memory Management

Effective memory management is a cornerstone of operating system design. In CertiKOS, physical memory is managed in page units (4KB each), ensuring each page can be individually allocated and protected. The physical memory management module in CertiKOS helps the kernel track which parts of memory are free, allocated, or reserved by other system components, enabling safe and efficient memory allocation and release.

#### Purpose and Structure

The memory management system in CertiKOS is divided into three layers, each serving a specific function in the memory allocation process:

* **MATINTRO Layer**: This introductory layer establishes a data structure to record the allocation and permission status of each physical memory page. Functions in this layer include basic getters and setters for the physical allocation table.
* **MATINIT Layer**: This initialization layer determines the physical memory's layout by retrieving information on available and reserved memory regions from the BIOS. It sets up an allocation table based on available memory and the kernel's reserved address ranges.
* **MATOP Layer**: The operational layer handles memory allocation and deallocation at the page level. It uses data from the previous layers to allocate pages on demand and release them when no longer needed.

Together, these layers provide a complete solution for managing physical memory at a low level within the CertiKOS kernel. Each layer relies on abstractions from the previous layers, ensuring modularity and ease of debugging.

#### Objectives of This Documentation

This documentation aims to provide:

1. **Explanation of the Core Components**: An in-depth look into the MATINTRO, MATINIT, and MATOP layers.
2. **Code Walkthroughs and Examples**: Explanations of key functions and data structures in each layer.
3. **Guidance for Implementation and Testing**: Tips on setting up and validating your implementations using the provided tests.

This guide assumes familiarity with C programming, basic operating system concepts, and x86 assembly. By the end, you should have a comprehensive understanding of how CertiKOS handles physical memory allocation and management, allowing you to efficiently implement and document similar features in other operating systems.
