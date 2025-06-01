+++
date = '2025-03-01T21:41:13+05:30'
draft = true
title = "Tokens: The Language of LLM's"
summary = "LLM's directly dont understand the human language well. It needs to be transfered to some form(token) for it to understand the language"
toc = true
readTime = true
autonumber = true
+++

## What are tokens? 
The input text given to a LLM is broken down into tokens which have numerical representation that can easily be understood by LLM.
Tokens are basically a collection of chars. The length of a token can range from a single char to couple of chars. 
for example the below sentence has `53` tokens and `252` characters
```
Many words map to one token, but some don't: indivisible.

Unicode characters like emojis may be split into many tokens containing the underlying bytes: 🤚🏾

Sequences of characters commonly found next to each other may be grouped together: 1234567890
```
you can try generating tokens form text using this [site](https://platform.openai.com/tokenizer)

A helpful rule of thumb is that one token corresponds to 4 chars of english text.

## Tokenization

The process of converting text to tokens is called tokenization. 