# Project Charter: SMB Paperless Log Book

## 1. Project Overview
The **SMB Paperless Log Book** is a digital solution designed to replace manual, paper-based logging systems in small to medium-sized businesses. It allows users to build custom logbooks (forms) through a web-based configurator, publish them, and collect data digitally.

## 2. Objectives
- **Digital Transformation**: Transition from physical logbooks to a digital, searchable database.
- **Customizability**: Provide a flexible "Form Builder" (Configurator) similar to Google Forms.
- **Accessibility**: Simple web interface for easy data entry on various devices.
- **Reliability**: Secure data storage using PostgreSQL and a robust FastAPI backend.

## 3. Scope
### In-Scope
- **Form Configurator**: UI to add fields (text, number, date, dropdown, signature).
- **Advanced Component Registry**: Rich schema support for validation, events (debounce), and data binding.
- **Visual Canvas Builder**: Grid-based "snap-to-grid" interface for field positioning (Low-code experience).
- **Form Publishing**: Generate a unique URL/access point for each logbook.
- **Data Entry Interface**: Dynamic rendering of the configured form for end-users.
- **Data Management**: View, filter, and potentially export logged entries.
- **Multi-tenant Architecture**: Support for multiple logbook types within the SMB environment.

### Out-of-Scope
- Integration with external ERPs (v1).
- Advanced AI-based data analytics (v1).

## 4. Technology Stack
- **Frontend**: Vanilla HTML5, CSS3 (Modern/Premium aesthetics), and JavaScript (ES6+).
- **Backend**: Python FastAPI (High performance, async).
- **Database**: PostgreSQL (Relational data for structured logs).
- **Infrastructure**: Triple-Stack Dockerized environment:
    - **Development**: `dev.smb.xelify.in` (Path: `/srv/xelify/dev.smb`)
    - **Staging**: `stage.smb.xelify.in` (Path: `/srv/xelify/stage.smb`)
    - **Production**: `smb.xelify.in` (Path: `/srv/xelify/smb`)

## 5. Stakeholders
- **System Administrator**: Manages the infrastructure and core settings.
- **Process Manager**: Creates and publishes logbooks using the Configurator.
- **Field Operator**: Enters data into the published logbooks.
