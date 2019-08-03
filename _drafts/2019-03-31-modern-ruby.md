---
layout: post
title:  Small things in modern Ruby that make me happy
date:   2019-03-31
categories: ruby
comments: true
---

**Preface:** Doesn't meant to be compendium of obscure tricks, but less known features with deep roots and consequences

The precious part of Ruby style

Not everybody fond of it, but that in fact the most distinctive

Parameters unpacking

`nil` + `&.` -- Ruby's `Maybe` monad

then
  two subjects in line
subject { post(base_url, params: params).then { response } }