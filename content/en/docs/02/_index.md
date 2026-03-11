---
title: "2. Setting up the Backstage Catalog"
weight: 2
sectionnumber: 2
---

The Backstage Software Catalog is the heart of your developer portal. It provides a centralized view of all software components, services, APIs, resources, and teams in your organization. In this chapter, you'll learn how to populate and manage the catalog effectively.

## Understanding the Catalog

The catalog uses YAML files to describe entities. Each entity represents something in your software ecosystem:

- **Components**: Individual pieces of software (services, libraries, websites)
- **APIs**: Interfaces that components provide
- **Resources**: Infrastructure resources (databases, queues, storage)
- **Systems**: Collections of components and resources
- **Domains**: Groups of related systems
- **Groups**: Teams or organizational units
- **Users**: Individual people

{{% alert title="Note" color="primary" %}}
The catalog uses a declarative approach - you describe what exists, and Backstage takes care of displaying and organizing it.
{{% /alert %}}


## Task {{% param sectionnumber %}}.1: Create Your First Component

Let's create a simple microservice component and register it in the catalog.

Create a new sub-directory for your sample service into the examples folder:

```bash
mkdir -p examples/my-sample-service
```

Create a `catalog-info.yaml` file with the following content:

```yaml
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: my-sample-service
  description: A sample microservice for the Backstage catalog
  annotations:
    github.com/project-slug: your-org/my-sample-service
  tags:
    - nodejs
    - microservice
  links:
    - url: https://dashboard.example.com
      title: Service Dashboard
      icon: dashboard
spec:
  type: service
  lifecycle: production
  owner: team-a
  system: my-system
```

**Understanding the structure:**
- `metadata.name`: Unique identifier for the component
- `metadata.description`: Human-readable description
- `metadata.annotations`: Additional metadata (like GitHub repository)
- `metadata.tags`: Labels for filtering and searching
- `spec.type`: Type of component (service, library, website, etc.)
- `spec.lifecycle`: Stage of development (experimental, production, deprecated)
- `spec.owner`: Team or group that owns this component

See the [Backstage Descriptor Format](https://backstage.io/docs/features/software-catalog/descriptor-format/) for more details.

## Task {{% param sectionnumber %}}.2: Register the Component in Backstage

Now let's register this component in your Backstage instance.

**Option 1: Register via URL (if using Git)**

If you've pushed this file to a Git repository:

1. Navigate to your Backstage instance at `http://localhost:3000`
2. Click on "Create..." in the sidebar
3. Click "Register Existing Component"
4. Enter the URL to your `catalog-info.yaml` file
5. Click "Analyze" and then "Import"

**Option 2: Register via Local File**

For local development, add the location to your `app-config.yaml`:

```yaml
catalog:
  locations:
    - type: file
      target: ../../examples/my-sample-service/catalog-info.yaml
```

After saving, Backstage will automatically pick up the new component. Navigate to the Catalog to see your component!


## Task {{% param sectionnumber %}}.3: Create a Complete System

Let's create a more complex example with multiple components forming a system.

Create a new file `catalog-system.yaml`:

```yaml
---
apiVersion: backstage.io/v1alpha1
kind: System
metadata:
  name: my-system
  description: An e-commerce platform system
  tags:
    - ecommerce
spec:
  owner: team-a
  domain: retail
---
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: frontend-app
  description: Customer-facing web application
  tags:
    - react
    - frontend
spec:
  type: website
  lifecycle: production
  owner: team-a
  system: my-system
  providesApis:
    - user-api
---
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: backend-api
  description: Backend REST API service
  tags:
    - nodejs
    - api
spec:
  type: service
  lifecycle: production
  owner: team-a
  system: my-system
  providesApis:
    - user-api
    - order-api
  consumesApis:
    - payment-api
---
apiVersion: backstage.io/v1alpha1
kind: API
metadata:
  name: user-api
  description: User management API
spec:
  type: openapi
  lifecycle: production
  owner: team-a
  system: my-system
  definition: |
    openapi: 3.0.0
    info:
      title: User API
      version: 1.0.0
    paths:
      /users:
        get:
          summary: List all users
          responses:
            '200':
              description: Successful response
---
apiVersion: backstage.io/v1alpha1
kind: Resource
metadata:
  name: user-database
  description: PostgreSQL database for user data
spec:
  type: database
  lifecycle: production
  owner: team-a
  system: my-system
```

Register this file in your `app-config.yaml`:

```yaml
catalog:
  locations:
    - type: file
      target: ../../catalog-system.yaml
```

{{% alert title="Note" color="primary" %}}
Notice how components reference each other through `providesApis` and `consumesApis`. This creates a dependency graph that Backstage visualizes automatically.
{{% /alert %}}


## Task {{% param sectionnumber %}}.4: Define Teams and Ownership

Ownership is crucial for accountability. Let's define teams in the catalog.

Create a `catalog-org.yaml` file:

```yaml
---
apiVersion: backstage.io/v1alpha1
kind: Group
metadata:
  name: team-a
  description: Platform Engineering Team
spec:
  type: team
  profile:
    displayName: Platform Engineering
    email: platform@example.com
  children: []
  members:
    - john.doe
    - jane.smith
---
apiVersion: backstage.io/v1alpha1
kind: User
metadata:
  name: john.doe
  description: Senior Platform Engineer
spec:
  profile:
    displayName: John Doe
    email: john.doe@example.com
  memberOf:
    - team-a
---
apiVersion: backstage.io/v1alpha1
kind: User
metadata:
  name: jane.smith
  description: Platform Engineer
spec:
  profile:
    displayName: Jane Smith
    email: jane.smith@example.com
  memberOf:
    - team-a
```

Register this in your `app-config.yaml`:

```yaml
catalog:
  locations:
    - type: file
      target: ../../catalog-org.yaml
```

Now when you view components, you'll see the actual team members who own them!


## Task {{% param sectionnumber %}}.5: Use Catalog Processors

Backstage can automatically discover and import entities from various sources. Let's configure GitHub discovery.

Edit your `app-config.yaml` to add GitHub integration:

```yaml
integrations:
  github:
    - host: github.com
      token: ${GITHUB_TOKEN}

catalog:
  providers:
    github:
      myOrg:
        organization: 'your-github-org'
        catalogPath: '/catalog-info.yaml'
        filters:
          branch: 'main'
          repository: '.*'
        schedule:
          frequency: { minutes: 30 }
          timeout: { minutes: 3 }
```

{{% alert title="Note" color="primary" %}}
You'll need to set the `GITHUB_TOKEN` environment variable with a personal access token that has read access to your repositories.
{{% /alert %}}

This configuration will automatically discover all repositories in your GitHub organization that contain a `catalog-info.yaml` file!


## Task {{% param sectionnumber %}}.6: Add TechDocs to Your Component

TechDocs brings documentation directly into Backstage. Let's add documentation to one of your components.

In your `my-sample-service` directory, create a `docs/` folder:

```bash
mkdir -p ~/my-sample-service/docs
```

Create `docs/index.md`:

```markdown
# My Sample Service

## Overview

This is a sample microservice that demonstrates Backstage catalog integration.

## Architecture

The service is built with Node.js and provides a REST API for user management.

## Getting Started

### Prerequisites

- Node.js 18+
- PostgreSQL 14+

### Installation

\`\`\`bash
npm install
npm start
\`\`\`

## API Documentation

See the [API Reference](./api.md) for detailed endpoint documentation.
```

Update your `catalog-info.yaml` to enable TechDocs:

```yaml
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: my-sample-service
  description: A sample microservice for the Backstage catalog
  annotations:
    github.com/project-slug: your-org/my-sample-service
    backstage.io/techdocs-ref: dir:.
  tags:
    - nodejs
    - microservice
spec:
  type: service
  lifecycle: production
  owner: team-a
  system: my-system
```

Create a `mkdocs.yml` file in the root of your service:

```yaml
site_name: 'My Sample Service'
site_description: 'Documentation for My Sample Service'

nav:
  - Home: index.md

plugins:
  - techdocs-core
```

Now your documentation will be available directly in Backstage under the "Docs" tab of your component!


## Task {{% param sectionnumber %}}.7: Explore Catalog Relationships

Navigate to your Backstage catalog and explore the relationships:

1. Go to `http://localhost:3000/catalog`
2. Click on the `backend-api` component
3. Explore the different tabs:
   - **Overview**: Basic information
   - **Dependencies**: What this component depends on
   - **API**: APIs provided by this component
   - **Docs**: TechDocs documentation (if configured)

Notice the visual dependency graph showing how components relate to each other!


## Best Practices for Catalog Management

As you build out your catalog, keep these best practices in mind:

1. **Keep catalog files with the code**: Store `catalog-info.yaml` in the same repository as the component
2. **Use consistent naming**: Follow a naming convention (e.g., kebab-case)
3. **Tag appropriately**: Use tags for technology, team, and purpose
4. **Define clear ownership**: Every component should have an owner
5. **Document relationships**: Use `dependsOn`, `providesApis`, and `consumesApis`
6. **Keep it up to date**: Automate catalog updates through CI/CD
7. **Use systems and domains**: Group related components for better organization


## Summary

In this chapter, you:
- ✅ Created your first catalog component
- ✅ Registered components in Backstage
- ✅ Built a complete system with multiple entities
- ✅ Defined teams and ownership
- ✅ Configured automatic discovery from GitHub
- ✅ Added TechDocs documentation
- ✅ Explored catalog relationships

Your Backstage catalog is now populated with meaningful data that represents your software ecosystem!  
