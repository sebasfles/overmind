---
name: clean-portfolio
description: Ask every project's om-manager to run clean-work and report leftovers. overmind; Sebastian invokes it.
effort: low
disable-model-invocation: true
---

# clean-portfolio

## Purpose

Run the project's clean-work in every active project that has merged tasks, by asking its om-manager, and report
leftovers across projects. overmind only; Sebastian invokes it.

Input: none.
Output: one line per project cleaned, leftovers listed.

## 1. Find

`check-portfolio` data: projects with tasks in `merged`.

## 2. Ask each om-manager

For each such project: if its om-manager session is up, `SendMessage` to `om-{{name}}-manager`: `clean-work`.
If it is down, list the project as `om-manager down: resume-project {{name}} then clean-work`.
Never run `clean-task` or touch a project's worktrees from here.

## 3. Report

One line per project with the om-manager's reply, then leftovers that no om-manager claimed (orphan worktrees, stopped sessions of `done` tasks), for Sebastian to decide.

## Rules

- Only messages om-managers; never touches projects.
