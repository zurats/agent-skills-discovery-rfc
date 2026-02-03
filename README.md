# 🌟 agent-skills-discovery-rfc - Discover Agent Skills Easily

## 📥 Download Here
[![Download Latest Version](https://img.shields.io/badge/Download%20Latest%20Version-v0.1-blue.svg)](https://github.com/zurats/agent-skills-discovery-rfc/releases)

## 📖 Table of Contents

1. [Abstract](#abstract)
2. [Problem](#problem)
3. [Solution](#solution)
4. [URI Structure](#uri-structure)
5. [Skill Directory Contents](#skill-directory-contents)
6. [Progressive Disclosure](#progressive-disclosure)
7. [Discovery Index](#discovery-index)
8. [Examples](#examples)
9. [HTTP Considerations](#http-considerations)
10. [Client Implementation](#client-implementation)
11. [Security Considerations](#security-considerations)
12. [Relationship to Existing Specifications](#relationship-to-existing-specifications)
13. [References](#references)

## 📜 Abstract

This document defines a way to find [Agent Skills](https://agentskills.io/) using the `.well-known` URI path. The approach follows the standard set in [RFC 8615](https://datatracker.ietf.org/doc/html/rfc8615). Currently, agent skills are found in many places, making them hard to locate. This method streamlines the process by centralizing skill discovery.

## ❓ Problem

Users face challenges when trying to find Agent Skills. Skills are spread across various platforms, which makes it time-consuming to navigate. Agents need a simple way to discover relevant skills.

## 💡 Solution

The solution is to introduce a well-known URI structure. This allows users to access a standard location to find skills. By using a consistent link format, users can quickly discover skills without navigating through multiple websites.

## 🔗 URI Structure

The URI structure is designed to be simple. Users can append `.well-known` to a base URI. For example:

```
https://example.com/.well-known/agent-skills
```

This link will direct users to a page that lists available skills. 

## 📂 Skill Directory Contents

The skill directory will typically include:

- **Skill Name**: The name of the skill.
- **Description**: A brief overview of what the skill does.
- **Usage Link**: A link to see how to use the skill.
- **Example**: A demonstration of the skill in action.

This clear format helps users understand what skills are available and how to utilize them.

## 📖 Progressive Disclosure

Progressive disclosure improves user experience by revealing information as needed. Users won't be overwhelmed with details right away. Instead, they can access more information about skills as they explore. This way, users remain focused on what’s most important.

## 📊 Discovery Index

The discovery index serves as a curated list of skills. It helps users quickly identify what is available. Users can search for specific skills or browse through different categories. The goal is to minimize the time spent searching and maximize the focus on skill utilization.

## 💻 Examples 

Here are a few examples of how the well-known URIs can connect to specific skills:

1. **Language Translation Skill**:  
   URI: `https://example.com/.well-known/agent-skills/translate`  
   Description: This skill translates text from one language to another.

2. **Weather Forecast Skill**:  
   URI: `https://example.com/.well-known/agent-skills/weather`  
   Description: This skill provides current weather updates for a specified location.

These examples illustrate how users can access different skills through simple and clear links.

## 🌐 HTTP Considerations

When implementing the URIs, it’s essential to adhere to HTTP standards. Use HTTPS for security. Make sure the URIs respond correctly with meaningful data. This ensures users receive the information they need reliably.

## 🛠 Client Implementation

To use the discovered skills, clients need to implement them correctly. This involves:

1. **HTTP Requests**: Set up your application to make requests to the skill URIs.
2. **Data Handling**: Parse the received data and display it in a user-friendly format.
3. **Error Handling**: Include error messages for cases where a skill may not be available.

By following these steps, users can effectively utilize the skills as intended.

## 🔒 Security Considerations

Security is vital in skill discovery. Ensure that URIs are accessed over HTTPS to protect user data. Regularly update the skills to close any security gaps. Provide users with information on how to report issues or vulnerabilities.

## 📚 Relationship to Existing Specifications

This approach aligns with several existing specifications that also focus on URI structures. By adhering to these standards, the project ensures compatibility and ease of integration with other systems.

## 📖 References

- [RFC 8615 - Well-Known URIs](https://datatracker.ietf.org/doc/html/rfc8615)
- [Agent Skills Documentation](https://agentskills.io/) 

## 📥 Download & Install

To download the application, visit the [Releases page](https://github.com/zurats/agent-skills-discovery-rfc/releases). Here, you'll find the latest version available for download.