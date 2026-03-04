---
name: codecheck
description: analyze and fix code format/lint error
disable-model-invocation: true
---

# Code Check

For files affected in current commit, analyze code format and lint error with the provided sciprts, then fix them.

## Usage

Run the python script: `python .agents/skills/codecheck/scripts/ms_codecheck.py` to analzye lint and format error. 

Format the code according to the script output. 

cpplint may be contradictory to clangformat. In such case, add filters in .jenkins/check/config.
Only add filters after obtaining permission.

After formatting, run the script again to make sure there's no more error anymore.
