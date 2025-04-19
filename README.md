# HarmonySpace: A High-Level Language for Agent Systems

HarmonySpace is a declarative domain-specific language (DSL) for designing and implementing agent-based systems based on agent frameworks such as LangGraph. It provides a structured way to define agent interactions, information flows, and coordination patterns using a YAML-like syntax.

## Overview

Building complex agent systems using high-level frameworks like LangGraph requires a significant amount of boilerplate code and a deep understanding of the framework.  HarmonySpace abstracts away these details, allowing developers to focus on the high-level design of their agent systems.

The language is designed around the concept of "harmony spaces" - environments where agents collaborate through well-defined information flows and coordination mechanisms.

## Features

- **Declarative Syntax**: Define systems using a clean, readable YAML-like syntax
- **Agent-Centric Design**: Agents are first-class citizens with definable capabilities, knowledge, and behavior
- **Flow-Based Programming**: Define information flows between agents and components
- **Human Integration**: First-class support for human-in-the-loop processes
- **Composability**: Build complex systems from reusable components

## Core Concepts

HarmonySpace is based on the key concepts of graph-based agent interaction, while adding high-level abstractions:

- **Agents**: Autonomous components that process information and perform tasks
- **Flows**: Defined paths of information and control between agents
- **State**: Structured data that flows through the system
- **Human Integration**: Points where human intervention is required or possible
- **Subgraphs**: Reusable components that encapsulate functionality

## Documentation

- [Language Specification](harmony_lang_spec.md): Detailed language specification
- [Examples](harmony_examples.md): Example systems built with HarmonySpace
- [Compiler Design](harmony_compiler.md): Design of the HarmonySpace compiler

