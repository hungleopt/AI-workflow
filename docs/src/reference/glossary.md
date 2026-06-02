# Glossary

## DXP

Optimizely Digital Experience Platform, the cloud hosting target used by the backend deployment pipeline.

## Optimizely CMS

The CMS framework used by `firstmile.web` and `FirstMile.Models` for content types, routing, editor behavior, and page rendering.

## Pattern Lab

The component and static frontend build system used in `firstmile.ui`.

## Release package branch

A generated branch created by the frontend integration pipeline to carry built frontend artifacts back into a backend-target branch through a pull request.

## Feature folder

A web project structure that groups presentation logic by business area rather than by technical MVC folder type.

## Request context

A scoped service registered as `IRequestContext` and used to carry request-level state through the application.

## Primary domain

The public production domain identified in `DomainConstants` as `www.thefirstmile.co.uk`, used by startup logic for indexing behavior.
