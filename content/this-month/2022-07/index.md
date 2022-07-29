+++
title = "This Month in Rust OSDev: July 2022"
date = 0000-01-01

[extra]
month = "July 2022"
authors = [
    "phil-opp",
    "ColinFinck",
    # add yourself here
]
+++

Welcome to a new issue of _"This Month in Rust OSDev"_. In these posts, we give a regular overview of notable changes in the Rust operating system development ecosystem.

<!-- more -->

This series is openly developed [on GitHub](https://github.com/rust-osdev/homepage/). Feel free to open pull requests there with content you would like to see in the next issue. If you find some issues on this page, please report them by [creating an issue](https://github.com/rust-osdev/homepage/issues/new) or using our [_comment form_](#comment-form) at the bottom of this page.

<!--
    This is a draft for the upcoming "This Month in Rust OSDev (July 2022)" post.
    Feel free to create pull requests against the `next` branch to add your
    content here.
    Please take a look at the past posts on https://rust-osdev.com/ to see the
    general structure of these posts.
-->

## Project Updates

In this section, we give an overview of notable changes to the projects hosted under the [`rust-osdev`] organization.

[`rust-osdev`]: https://github.com/rust-osdev/about

### [`x86_64`](https://github.com/rust-osdev/x86_64)

<span class="maintainers">Maintained by [@phil-opp](https://github.com/phil-opp), [@josephlr](https://github.com/orgs/rust-osdev/people/josephlr), [@Freax13](https://github.com/orgs/rust-osdev/people/Freax13), and [@rybot666](https://github.com/orgs/rust-osdev/people/rybot666)</span>

The `x86_64` crate provides various abstractions for `x86_64` systems, including wrappers for CPU instructions, access to processor-specific registers, and abstraction types for architecture-specific structures such as page tables and descriptor tables.

In July, …

## Call for Participation

Want to contribute to a Rust OSDev project, but don't know where to start? Pick up one of these outstanding
issues in one of our projects and get started!

<!--
Please use the following template for adding items:
- [(`repo_name`) Issue Description](https://example.com/link-to-issue)
-->

<span class="gray">

_No tasks were proposed for this section._

</span>

If you maintain a Rust project related to operating system development and are looking for contributors, especially for tasks suited to people getting started in this space, please [create a PR](https://github.com/rust-osdev/homepage/pulls) against the `next` branch with the tasks you want to include in the next issue.

## Other Projects

In this section, we describe updates to Rust OS projects that are not directly related to the `rust-osdev` organization. Feel free to [create a pull request](https://github.com/rust-osdev/homepage/pulls) with the updates of your OS project for the next post.

### Comparison between [`phip1611/simple-chunk-allocator`](https://github.com/phip1611/simple-chunk-allocator) and [`rust-osdev/linked-list-allocator`](https://github.com/rust-osdev/linked-list-allocator)

<span class="gray">(Section written by [@phip1611](https://github.com/phip1611))</span>

In March 2022, Philipp Schuster proposed his [`simple-chunk-allocator`](https://github.com/phip1611/simple-chunk-allocator)
crate. It focuses on being a very simple-to-use general purpose allocator that "just works" for various workloads
in `no_std` context. However, there are other allocators, such as [`rust-osdev/linked-list-allocator`](https://github.com/rust-osdev/linked-list-allocator).
When you choose an allocator, you should not only consider the API and how to set it up, but actually the runtime 
characteristics. OS research has shown us that there is no perfect allocator. You can optimize an allocator for speed,
memory utilization (i.e., prevent fragmentation), code simplicity, and worst case execution time. There exist different
strategies to reach those goals: first-fit, next-fit, best-fit

Recently, Philipp benchmarked his `simple-chunk-allocator` against [`rust-osdev/linked-list-allocator`](https://github.com/rust-osdev/linked-list-allocator) 
to learn under which conditions which performs better. But at first, let's point out some differences. `simple-chunk-allocator` needs 
a static backing storage for heap and an additional static backing storage for its internal bookkeeping. `linked-list-allocator`
can solve this better by organizing the heap with the heap backing memory itself. `simple-chunk-allocator` uses a slightly
adjusted variant of best-fit that tries to reduce fragmentation. `linked-list-allocator` is a first-fit allocator that 
has a lower performance to more heap is used.

The relevant outcome is that `simple-chunk-allocator` always outperforms `linked-list-allocator` in median allocation time.
For stress tests with a low heap consumption, thus, no long stress test with 90% and more heap usage, `simple-chunk-allocator`
also outperforms `linked-list-allocator` in average allocation time. However, if the heap is full and frequent allocations
happen, the average (but not the median) allocation time of `linked-list-allocator` is better. Also, `linked-list-allocator`
can come close to 100% heap usage which is not the case for `simple-chunk-allocator`, because it suffers from internal
fragmentation under certain circumstances. Last but not least, even small allocations always takes up a multiple of the
used chunk size in `simple-chunk-allocator`.

In the end, there is no optimal allocator. You must choose which properties are more relevant for your scenario.
For concrete measurements, please head to the README of [`simple-chunk-allocator`](https://github.com/phip1611/simple-chunk-allocator).

### [`nt-list`: Windows Linked Lists in idiomatic Rust](https://colinfinck.de/posts/nt-list-windows-linked-lists-in-idiomatic-rust/)

<span class="gray">(Section written by [@ColinFinck](https://github.com/ColinFinck))</span>

On his quest to develop a ReactOS/Windows bootloader in Rust, Colin Finck released another building block as a reusable `no_std` crate this month.
After [nt-hive](https://github.com/ColinFinck/nt-hive) for reading Windows registry hive files and [ntfs](https://github.com/ColinFinck/ntfs) to access Microsoft's proprietary NTFS filesystem, the [nt-list](https://github.com/ColinFinck/nt-list) crate provides a type-safe and idiomatic Rust interface to work with Windows Linked Lists, known as [`LIST_ENTRY`](https://docs.microsoft.com/en-us/windows/win32/api/ntdef/ns-ntdef-list_entry) and [`SINGLE_LIST_ENTRY`](https://docs.microsoft.com/en-us/windows/win32/api/ntdef/ns-ntdef-single_list_entry).
This is what Windows, Windows drivers, and components influenced by Windows (e.g. UEFI) have been using for a long time to uniformly handle linked lists.

[Colin's blog post](https://colinfinck.de/posts/nt-list-windows-linked-lists-in-idiomatic-rust/) goes into detail about some of the differences between textbook and Windows linked lists and the challenges in coming up with a safe Rust implementation.
The final interface provided by nt-list is as simple to use as `Vec` while being fully compatible to the original `LIST_ENTRY`.
The compatibility is proven in a WinDbg debugging session:

[![Using WinDbg to traverse a Windows Linked List created by the nt-list Rust crate](windbg.png "Using WinDbg to traverse a Windows Linked List created by the nt-list Rust crate")](windbg.png)

If you want to give it a spin, the crate is available on [crates.io](https://crates.io/crates/nt-list), and make sure to also check the [docs](https://docs.rs/nt-list/).

### [`phil-opp/blog_os`](https://github.com/phil-opp/blog_os)

<span class="gray">(Section written by [@phil-opp](https://github.com/phil-opp))</span>

This month, ...

## Join Us?

Are you interested in Rust-based operating system development? Our `rust-osdev` organization is always open to new members and new projects. Just let us know if you want to join! A good way for getting in touch is our [gitter channel](https://gitter.im/rust-osdev/Lobby).


<!--
TODO: Update publication date
-->
