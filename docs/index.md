# About
GitGudSecurity is an information security knowledge base made from bunch of infosec stuff I read and compile when I have the will power.

Due to the my [workflow](#workflow), this is to be expected:

- stub article, broken link etc. Since not all locally stored notes are hosted here
- unclear source/ references. This was essentially a personal notes and references might not be very good.

These issues, in due time, will be fixed - but my take a while since this is a side project.

## Workflow

Here is the workflow

```mermaid
graph TD

feedly --> pocket
socmed[twitter/mastdn/bsky]  --> pocket
pocket --> | summarize | compendiums[compendiums@local KB] 
try[thoughts, experiments, experience] --> compendiums 
compendiums --> | fix markdown | GitGudSecurity
```