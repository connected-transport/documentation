# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This repository contains the **ITS-FMS Interface** documentation, which defines a standard for communication between logistic IT-suppliers (Fleet Management Systems) and ITS cloud service providers (CSP) that offer Intelligent Transport Systems services. These services provide real-time traffic and infrastructure events relevant to truck routes.

The interface is an implementation of the Open Trip Model 5.0 (OTM 5.0) standard and has been developed with and implemented by several market companies with support of the Dutch government.

## Repository Structure

```
documentation/
├── api/
│   └── its-fms.yaml          # Main OpenAPI 3.1.0 specification (3000+ lines)
├── assets/                    # Diagrams and images used in documentation
│   ├── intro.png
│   ├── figure_*.png           # Architecture and flow diagrams
│   └── table_*.png
├── index.html                 # Redocly viewer for the API spec
└── README.md                  # Basic repository introduction
```

## Documentation Workflow

### Viewing the Documentation

The documentation is published using Redocly. You can view it in two ways:

**Online (recommended):**
- Direct Redocly render: https://redocly.github.io/redoc/?url=https://raw.githubusercontent.com/connected-transport/documentation/main/api/its-fms.yaml
- Always shows the latest version from the main branch

**Locally:**
1. Open `documentation/index.html` in a browser (requires internet connection for Redocly CDN)
2. The HTML file references `api/its-fms.yaml` as the OpenAPI specification
3. Useful for previewing local changes before committing

### Working with the OpenAPI Specification

The main documentation file is `documentation/api/its-fms.yaml`. This is a comprehensive OpenAPI 3.1.0 specification that documents:

**Communication Types:**
- REST API (JSON over HTTPS) for static data (vehicle and route announcements)
- WebSocket (Protocol Buffers over WSS) for dynamic real-time data (events, positions, control messages)

**Key API Endpoints:**
- `PUT /vehicles` - Announce a truck to the CSP (required every 24 hours)
- `PUT /routes` - Announce the truck's route (required for iVRI priority use case)
- `DELETE /routes/{RouteId}` - Remove a completed or cancelled route
- WebSocket `/protobuf_message` - Real-time bidirectional communication

**Interface Flow:**
1. Announce vehicle via REST API
2. Announce route via REST API (only if iVRIPriorityEnabled is true)
3. Open WebSocket connection (one per vehicle)
4. Start communication by sending first location update
5. Respond to timestamp requests to keep connection alive
6. Receive real-time event messages from CSP

**Important Concepts:**
- The YAML file contains extensive inline documentation under `tags` sections covering:
  - High level architecture
  - Communication types (REST vs WebSocket/Protobuf)
  - Authentication (Bearer tokens)
  - Authorization (bilateral contracts)
  - Interface flow (6-step process)
  - Event codes (ITS use cases: traffic warnings, iVRI priority, etc.)
  - Dynamic sampling for location updates
  - GPS accuracy requirements
  - Inactivity monitoring (5-minute timeout)
  - Use case specific functionality

**ITS Event Codes:**
The interface supports various ITS use cases identified by event codes:
- **001**: Changed frequency request (dynamic sampling)
- **101-104**: Signaling and speed advice (max speed, advisory speed, overtaking ban, lane config)
- **201-212**: Dangerous situations (road works, traffic jam, incidents, weather, emergency vehicles)
- **301-302**: iVRI priority (priority status, selective access)
- **401-404**: iVRI current information (traffic light status/timing, optimal speed advice)
- **501-505**: Sensor data (tire pressure, vehicle height/weight, parking)

### Editing the Specification

When modifying `its-fms.yaml`:

1. **Maintain OpenAPI 3.1.0 compliance** - The spec uses advanced features like `allOf`, `$ref`, discriminators
2. **Preserve inline documentation** - Extensive descriptions are embedded in `tags[].description` fields using markdown
3. **Keep consistency** - The spec documents both REST and WebSocket/Protobuf interfaces, though WebSocket endpoints are documented as REST-like for readability
4. **Update version** - The spec version is in `info.version` (currently 2.1.0)
5. **Maintain asset references** - Images are referenced as `![Figure1](assets/filename.png)` in descriptions
6. **Test with Redocly** - Validate changes by viewing through `index.html`

### Key Schema Patterns

The YAML uses consistent patterns:
- **Vehicle schemas** include required fields: `id` (UUID), `vehicleType` (enum), `fuel` (enum)
- **externalAttributes** extend OTM with interface-specific properties:
  - `iVRIPriorityEnabled`: boolean to activate priority use case
  - `dynamicRate`: boolean for dynamic vs fixed (1 Hz) location updates
- **Message types** follow protobuf structure with `messageType` field and nested `container` objects
- **Event messages** have `infoContainer` (metadata) and `contentContainer` (event-specific data)

## Technology Stack

- **Format**: OpenAPI 3.1.0 / YAML
- **Documentation**: Redocly for rendering
- **Standards**: Based on Open Trip Model 5.0 (OTM)
- **Communication Protocols**:
  - REST API (JSON)
  - WebSocket (Protocol Buffers proto3)
- **Authentication**: Bearer tokens over TLS

## No Build or Test Commands

This is a documentation-only repository. There are no build, lint, or test commands. Changes are reviewed by viewing the rendered documentation through Redocly.

## External References

The specification references several external standards:
- [Open Trip Model documentation](https://www.opentripmodel.org) and [OTM 5.0 spec](https://otm5.opentripmodel.org/)
- [CROW standards on C-ITS](https://www.crow.nl/thema-s/verkeersmanagement/landelijke-ivri-standaarden)
- [Protocol Buffers documentation](https://developers.google.com/protocol-buffers)
- Demo implementations: [C#](https://github.com/svendrig/connected_transport/tree/main/c%23) and [Java](https://github.com/svendrig/connected_transport/tree/main/java)

## Contact Information

- **Maintainer**: Ministry of Infrastructure and Waterworks
- **Support**: support@movingit.nl
- **Server**: https://movingit.eu/ctc
