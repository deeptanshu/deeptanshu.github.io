---
title: "Claude Code Amazing Debug Where Chatgpt Failed"
date: 2026-01-28T00:46:05-08:00
draft: true
category: ""
excerpt: ""
tags: []
---

  Image Overflow Bug - Final Root Cause Analysis                                                                                      
                                                                                                                                      
  The Problem                                                                                                                         
                                                                                                                                      
  Images in blog posts were rendering at their native size (1500-3500px wide) and overflowing beyond the 850px content container.     
                                                                                                                                      
  The Actual Root Cause                                                                                                               
                                                                                                                                      
  A Unicode arrow character → in line 311 of site.css was breaking CSS parsing.                                                       
                                                                                                                                      
  /* This broke the CSS parser */                                                                                                     
  .essay-link::after { content: '→'; ... }                                                                                            
                                                                                                                                      
  /* Fixed version */                                                                                                                 
  .essay-link::after { content: '\2192'; ... }                                                                                        
                                                                                                                                      
  The browser received the full CSS file, but the CSS parser choked on the Unicode character and silently stopped parsing everything  
  after line 311. All rules after that point (including the .prose img rules at line 471) were ignored.                               
                                                                                                                                      
  Why ChatGPT Couldn't Fix It                                                                                                         
                                                                                                                                      
  1. Kept adding rules that were never parsed - It added 5 different .prose img rules, not realizing none of them were being applied  
  because parsing stopped earlier in the file                                                                                         
  2. Focused on CSS specificity instead of delivery - It kept adding !important flags and adjusting selectors, fighting a problem that
   didn't exist                                                                                                                       
  3. No interactive debugging - It couldn't ask you to check DevTools to verify rules were actually being applied                     
  4. Didn't investigate parsing failures - When CSS rules don't apply, the first question should be "is this rule reaching the browser
   and being parsed?" not "is my specificity wrong?"                                                                                  
                                                                                                                                      
  The Debugging Process That Found It                                                                                                 
                                                                                                                                      
  1. Added inline <style> with same selector → worked → proved selector was valid                                                     
  2. Checked Network tab Response → CSS file was delivered correctly                                                                  
  3. But rules from end of file weren't in Styles panel → parsing stopped mid-file                                                    
  4. Added test rule at end of file (yellow background) → not applied → confirmed parsing failure                                     
  5. Found Unicode → character → replaced with \2192 → test rule worked → fixed                                                       
                                                                                                                                      
  Lesson                                                                                                                              
                                                                                                                                      
  When CSS rules exist in your file but don't appear in DevTools, check for parsing errors earlier in the file. Unicode characters in 
  content properties are a common culprit.                                                                                            
