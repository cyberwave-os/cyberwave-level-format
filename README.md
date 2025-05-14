# Cyberwave Level Format Specification

Version: 1.0
Date: May 14, 2025
Document Status: Draft

## 1. Overview

The Cyberwave Level Format (CLF) is a YAML-based specification for defining digital twins, interactive 3D scenes, and immersive experiences that can be shared across gaming, cinematic, and industrial simulation platforms. This format provides a unified way to represent real-world environments, objects, and their behaviors in a web-first, cross-platform manner.

CLF is designed with the following key principles:

- **Web-first**: All resources are accessible via standard HTTPS URLs
- **Workspace-based**: Resources organized by tenant workspaces
- **Hierarchical**: Entity paths follow a consistent hierarchy
- **Shareable**: Scenes and components can be easily shared and embedded
- **Cross-platform**: Compatible with gaming engines, VFX tools, and industrial simulation systems

## 2. URI Structure

CLF uses a consistent URI pattern system to reference resources:

| Resource Type | URI Pattern |
|---------------|-------------|
| Catalog Asset | `https://cyberwave.com/catalog/{workspace}/{asset_id}` |
| Level Asset | `https://cyberwave.com/levels/{workspace}/{level_id}/assets/{asset_id}` |
| Catalog Stream | `https://cyberwave.com/streams/{workspace}/{stream_id}` |
| Level Stream | `https://cyberwave.com/levels/{workspace}/{level_id}/streams/{stream_id}` |
| Catalog Data | `https://cyberwave.com/data/{workspace}/{data_id}` |
| Level Data | `https://cyberwave.com/levels/{workspace}/{level_id}/data/{data_id}` |
| View | `https://cyberwave.com/view/{workspace}/{entity_path}` |

These URI patterns support both `https://` web access and proprietary `cyberwave://` protocol handlers in supporting applications.

## 3. Root Structure

A Cyberwave Level file must include the following top-level sections:

```yaml
metadata:      # Level metadata (required)
uri_patterns:  # URI pattern definitions (optional)
timeline:      # Time control (required)
path_system:   # Entity path configuration (required)
assets:        # Asset references (required)
streams:       # Data streams (optional)
entities:      # Entity hierarchy (required)
data_links:    # Data connections (optional)
behaviors:     # Entity behaviors (optional)
overlays:      # Visual overlays (optional)
queries:       # Data queries (optional)
sharing:       # Sharing configuration (optional)
```

## 4. Metadata

The `metadata` section contains core information about the level:

```yaml
metadata:
  name: string              # Display name
  version: string           # Semantic version
  author: string            # Creator name/org
  created: ISO8601 date     # Creation timestamp
  description: string       # Brief description
  coordinate_system: string # e.g., "right-handed, +Y up"
  usage_intent: string[]    # Intended platforms
  workspace_id: string      # Tenant identifier
  level_id: string          # Unique level identifier
  public: boolean           # Visibility flag
```

## 5. Path System

The `path_system` defines how entity paths are constructed and referenced:

```yaml
path_system:
  separator: "/"            # Path delimiter
  root: "world"             # Root namespace
  reserved_prefixes: string[] # Reserved namespaces
  workspace_prefix: boolean # Whether to prepend workspace
```

Paths use a hierarchy similar to a file system, e.g., `world/acme-factory/factory/machines/cnc_01`.

## 6. Assets

Assets are divided into catalog assets (shared across a workspace) and level-specific assets:

```yaml
assets:
  catalog:
    - id: string               # Asset identifier
      uri: string              # Web URI
      preview: string          # Preview image URL
      tags: string[]           # Categorization tags
      
  level:
    - id: string               # Asset identifier
      uri: string              # Web URI
      preview: string          # Preview image URL
      tags: string[]           # Categorization tags
```

### 6.1 Asset Types

CLF supports these common asset types:

- 3D Models: glTF/GLB, USD, OBJ, FBX, STEP
- Video: MP4, WebM, HLS streams
- Images: JPG, PNG, WebP, HDR
- Audio: MP3, WAV, OGG

The format type is inferred from the URI extension or explicitly specified in the asset definition.

## 7. Streams

Streams represent dynamic data flows, such as sensor values, trajectories, or video feeds:

```yaml
streams:
  catalog:
    stream_id:
      type: string          # Stream type
      uri: string           # Web URI
      description: string   # Description
      public: boolean       # Visibility flag
      real_time: boolean    # Live data flag
      
  level:
    stream_id:
      type: string          # Stream type
      uri: string           # Web URI
      target_path: string   # Target entity
      data_uri: string      # Data source
      public: boolean       # Visibility flag
      properties: object    # Stream-specific properties
```

### 7.1 Stream Types

| Type | Description |
|------|-------------|
| `articulation` | Movement trajectories for entity transforms |
| `parameters` | Named parameter values (temperature, RPM, etc.) |
| `point_cloud` | 3D point data (LiDAR, scans) |
| `video_frames` | Video stream frames |
| `audio` | Audio stream |

### 7.2 Reference Time

Most streams are linked to the level timeline. Real-time streams synchronize to wall clock time while maintaining relative timing with the timeline when paused or scrubbed.

## 8. Entities

Entities define the scene hierarchy, with each entity having a unique path:

```yaml
entities:
  - path: string           # Path segment
    asset_ref:             # Referenced asset
      type: "catalog"|"level"
      id: string
    transform: object      # Spatial transform
    web_url: string        # Web view URL
    streams: array         # Connected streams
    children: array        # Child entities
```

### 8.1 Transform

Transform defines the spatial properties of an entity:

```yaml
transform:
  pos: [x, y, z]           # Position vector
  rot_euler: [x, y, z]     # Rotation in degrees
  scale: number|[x, y, z]  # Uniform or vector scale
```

## 9. Data Links

Data links connect data streams to entity properties:

```yaml
data_links:
  - stream:                # Stream reference
      type: "catalog"|"level"
      id: string
    target_paths: string[] # Affected entity paths
    mapping:               # Data mapping
      stream_field: entity_property
```

Example mapping: `"temperature[0]": "spindle.temperature"` maps the first temperature value to the spindle's temperature property.

## 10. Behaviors

Behaviors define interactive or automated actions:

```yaml
behaviors:
  - id: string             # Behavior identifier
    type: string           # Behavior type
    steps: array           # For sequences
    condition: string      # For triggers
    actions: string[]      # Actions to execute
    web_url: string        # Web view URL
```

### 10.1 Behavior Types

| Type | Description |
|------|-------------|
| `sequence` | Timed sequence of actions |
| `trigger` | Condition-based actions |
| `interactive` | User-initiated actions |

### 10.2 Action Format

Actions use the format `entity_path.method`, optionally with a target: `entity_path.pick(target_path)`.

## 11. Overlays

Overlays provide visual representations of data or measurements:

```yaml
overlays:
  - id: string             # Overlay identifier
    type: string           # Overlay type
    data_source:           # Data stream
      type: "catalog"|"level"
      id: string
    target_path: string    # Applied to entity
    properties: object     # Type-specific properties
    web_url: string        # Web view URL
```

### 11.1 Overlay Types

| Type | Description |
|------|-------------|
| `heatmap` | Color-coded data visualization |
| `trajectory_visualization` | Path visualization |
| `measurement` | Distance or dimension display |
| `annotation` | Text or marker overlay |

## 12. Queries

Queries define how to extract and analyze data from the digital twin:

```yaml
queries:
  - id: string             # Query identifier
    metrics: string[]      # Values to measure
    target_path: string    # Source entity
    aggregation: string    # Aggregation method
    interval: number       # For time series
    dashboard_url: string  # Dashboard URL
    public: boolean        # Visibility flag
```

## 13. Sharing

The sharing section defines how the level can be distributed:

```yaml
sharing:
  embed_code: string       # HTML for embedding
  direct_link: string      # Direct web URL
  qr_code: string          # QR code URL
  social_preview:          # Preview metadata
    title: string
    description: string
    image_url: string
  access_control:          # Access settings
    type: string           # "public", "private", etc.
    allow_embedding: boolean
    domains_whitelist: string[]
```

## 14. Complete Example

See the full example in Appendix A for a comprehensive factory digital twin implementation.

## 15. Implementation Guidelines

### 15.1 Catalog vs. Level Assets

- Use catalog assets for components reused across multiple levels
- Use level assets for unique elements specific to one experience
- Reference with explicit type: `asset_ref: { type: "catalog", id: "robot_arm" }`

### 15.2 Workspace Isolation

Each workspace (tenant) maintains isolated assets, streams, and entities. Cross-workspace references require explicit permissions.

### 15.3 Web Compatibility

All elements should have web-accessible equivalents for sharing and collaboration. Web URLs follow the same pattern logic as the formal URIs.

### 15.4 Data Streaming

- Prerecorded streams use a data file with samples
- Real-time streams connect to WebSocket endpoints
- Mixed mode allows fallback to prerecorded when real-time unavailable

### 15.5 Path References

Always use full paths when linking between entities to ensure proper resolution across contexts.

---

## Appendix A: Complete Factory Digital Twin Example

See attached YAML file for a complete example implementing a factory digital twin with:
- Reusable catalog assets
- Level-specific customizations
- Sensor data streams
- Robot motion trajectories
- Automated behaviors
- Visual analytics

## Appendix B: Schema Definition

The formal JSON Schema definition for validation is available at:
`https://cyberwave.com/schemas/level-format/1.0.0/schema.json`

## Appendix C: Compatibility

| Platform | Import | Export | Live Editing | Streaming |
|----------|--------|--------|-------------|-----------|
| Unity | ✓ | ✓ | ✓ | ✓ |
| Unreal | ✓ | ✓ | ✓ | ✓ |
| Blender | ✓ | ✓ | - | - |
| Web | ✓ | - | - | ✓ |
| Rerun | ✓ | ✓ | - | ✓ |
| ROS2 | ✓ | ✓ | - | ✓ |
