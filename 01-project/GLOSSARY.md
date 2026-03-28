# Glossary

| Field         | Value                                |
|---------------|--------------------------------------|
| **Version**   | 1.0.0                                |
| **Status**    | Draft                                |
| **Author**    | Vox                                  |
| **Reviewer**  | Vox                                  |
| **Created**   | 2026-03-27                           |
| **Updated**   | 2026-03-27                           |

---

## 1. Purpose

This document defines terminology used across the **Utopia** project. All project documentation and code comments SHOULD use these terms consistently to avoid ambiguity.

## 2. Scope

All terms referenced in Utopia documentation, code comments, commit messages, and ADRs.

## 3. Terms

### A

| Term | Definition |
|------|-----------|
| **ADR** | Architecture Decision Record — a document capturing a significant architectural decision, its context, and consequences. |
| **ASVS** | Application Security Verification Standard — an OWASP framework defining security requirements for applications. |
| **ArgoCD** | A declarative GitOps continuous delivery tool for Kubernetes. |

### B

| Term | Definition |
|------|-----------|
| **Blue-Green Deployment** | A deployment strategy using two identical environments (blue and green) to enable zero-downtime releases. |
| **Bounded Context** | A Domain-Driven Design concept defining a clear boundary within which a domain model is defined and applicable. In Utopia, each module represents a bounded context. |

### C

| Term | Definition |
|------|-----------|
| **C4 Model** | A set of hierarchical architecture diagrams: Context, Container, Component, Code. |
| **Canary Deployment** | A deployment strategy that gradually rolls out changes to a small subset of users before full deployment. |
| **CI/CD** | Continuous Integration / Continuous Delivery — automated build, test, and deployment pipelines. |
| **Container** | A lightweight, standalone package of software that includes code, runtime, libraries, and settings. (Docker container) |
| **CORS** | Cross-Origin Resource Sharing — a browser security mechanism controlling which origins can access an API. |
| **CSP** | Content Security Policy — an HTTP header that restricts sources from which content can be loaded. |

### D

| Term | Definition |
|------|-----------|
| **DAST** | Dynamic Application Security Testing — security testing performed on a running application. |
| **DevSecOps** | An approach integrating security practices into the DevOps workflow at every stage. |
| **Domain Event** | An event representing something that happened in the domain. Used for inter-module communication in Utopia. |
| **DTO** | Data Transfer Object — an object used to transfer data between layers without exposing domain entities. |

### E

| Term | Definition |
|------|-----------|
| **EF Core** | Entity Framework Core — a .NET ORM for database access. |
| **Expand-Contract Pattern** | A database migration strategy: add new schema (expand), migrate data, remove old schema (contract). |

### F

| Term | Definition |
|------|-----------|
| **Fluent API** | EF Core's programmatic approach to configuring entity mappings, preferred over data annotations in Utopia. |
| **FluentValidation** | A .NET library for building strongly-typed validation rules. |

### G

| Term | Definition |
|------|-----------|
| **GitOps** | An operational framework using Git as the single source of truth for declarative infrastructure and workloads. |
| **Gitleaks** | A tool for detecting hardcoded secrets in Git repositories. |

### H

| Term | Definition |
|------|-----------|
| **Helm** | A package manager for Kubernetes that uses charts to define, install, and manage applications. |
| **Helm Chart** | A collection of Kubernetes manifest templates packaged for deployment via Helm. |
| **Harbor** | An open-source container registry with security scanning and policy-based image management. |

### I

| Term | Definition |
|------|-----------|
| **IaC** | Infrastructure as Code — managing infrastructure through machine-readable definition files. |
| **ISMS** | Information Security Management System — a systematic approach to managing sensitive information (ISO 27001). |

### K

| Term | Definition |
|------|-----------|
| **K3s** | A lightweight Kubernetes distribution designed for resource-constrained environments. |
| **K3d** | A tool for running K3s clusters inside Docker containers. |
| **Keycloak** | An open-source Identity and Access Management (IAM) solution providing OAuth 2.0 and OpenID Connect. |

### L

| Term | Definition |
|------|-----------|
| **Loki** | A log aggregation system by Grafana Labs, designed for storing and querying logs. |
| **Least Privilege** | A security principle granting only the minimum permissions necessary to perform a task. |

### M

| Term | Definition |
|------|-----------|
| **MassTransit** | A .NET distributed application framework providing message-based communication. |
| **Minimal API** | A simplified approach to building HTTP APIs in ASP.NET Core without controllers. |
| **Module (Utopia)** | A self-contained feature area within the Modulith, with its own domain, application, infrastructure, and API layers. |
| **Modulith** | Modular Monolith — a monolithic application designed with clear module boundaries that can be independently developed and potentially extracted into microservices. |
| **mTLS** | Mutual TLS — authentication where both client and server verify each other's certificates. |

### N

| Term | Definition |
|------|-----------|
| **Network Policy** | A Kubernetes resource that controls traffic flow between pods at the network level. |

### O

| Term | Definition |
|------|-----------|
| **OIDC** | OpenID Connect — an identity layer on top of OAuth 2.0 for user authentication. |
| **OPA** | Open Policy Agent — a policy engine for unified policy enforcement across the stack. |
| **OpenTelemetry (OTel)** | A set of APIs, SDKs, and tools for instrumenting, generating, and collecting telemetry data (metrics, logs, traces). |
| **OWASP** | Open Worldwide Application Security Project — a nonprofit foundation for improving software security. |

### P

| Term | Definition |
|------|-----------|
| **Pod** | The smallest deployable unit in Kubernetes, consisting of one or more containers. |
| **Prometheus** | A monitoring and alerting toolkit for collecting and querying time-series metrics. |

### Q

| Term | Definition |
|------|-----------|
| **Quality Gate** | A set of conditions that code must pass before it can proceed to the next stage in the pipeline. |

### R

| Term | Definition |
|------|-----------|
| **RBAC** | Role-Based Access Control — a method of restricting access based on user roles. |
| **Result Pattern** | A pattern returning a result object (success/failure) instead of throwing exceptions for expected failures. |
| **Runbook** | An operational document with step-by-step procedures for handling specific incidents or tasks. |

### S

| Term | Definition |
|------|-----------|
| **SAST** | Static Application Security Testing — analyzing source code for security vulnerabilities without executing it. |
| **SCA** | Software Composition Analysis — identifying known vulnerabilities in third-party dependencies. |
| **SemVer** | Semantic Versioning — a versioning scheme using MAJOR.MINOR.PATCH format. |
| **Sealed Secrets** | A Kubernetes controller and tool for one-way encrypted secrets. |
| **Shared Kernel** | Code shared across modules (e.g., base entities, common interfaces). Kept minimal in Utopia. |
| **SonarQube** | A platform for continuous code quality and security inspection. |

### T

| Term | Definition |
|------|-----------|
| **TDE** | Transparent Data Encryption — encrypting database files at rest without application changes. |
| **Tempo** | A distributed tracing backend by Grafana Labs. |
| **Terraform** | An IaC tool for provisioning and managing cloud infrastructure declaratively. |
| **Testcontainers** | A library providing lightweight, throwaway instances of databases and services for integration testing. |
| **Traefik** | A cloud-native reverse proxy and load balancer often used as a Kubernetes ingress controller. |
| **Trivy** | A comprehensive security scanner for containers, file systems, and dependencies. |
| **Trunk-Based Development** | A branching strategy where developers integrate small, frequent changes into a single main branch. |

### V

| Term | Definition |
|------|-----------|
| **Vault** | HashiCorp Vault — a tool for securely storing and accessing secrets, encryption keys, and certificates. |

### Z

| Term | Definition |
|------|-----------|
| **Zero-Trust** | A security model that requires strict identity verification for every person and device, regardless of network location. |
| **ZAP** | Zed Attack Proxy — an OWASP tool for automated DAST scanning. |

## 4. References

- [DOCUMENTATION-STANDARD.md](../00-standards/DOCUMENTATION-STANDARD.md)
- [TECH-STACK-DECISION.md](./TECH-STACK-DECISION.md)

## Changelog

| Version | Date       | Author | Description          |
|---------|------------|--------|----------------------|
| 1.0.0   | 2026-03-27 | Vox    | Initial draft        |
