---
title: "3. Using Software Templates"
weight: 3
sectionnumber: 3
---

Software Templates in Backstage enable self-service for developers. Instead of copying boilerplate code or following lengthy setup guides, developers can use templates to scaffold new projects with best practices built in. This chapter will teach you how to create and use templates effectively.

## Understanding Software Templates

Software Templates (also called Scaffolder Templates) allow you to:

- **Standardize project setup**: Ensure all projects follow organizational best practices
- **Reduce onboarding time**: New developers can create production-ready projects in minutes
- **Enforce compliance**: Build in security, monitoring, and governance from day one
- **Accelerate development**: Eliminate repetitive setup tasks

{{% alert title="Note" color="primary" %}}
Templates use a declarative YAML format and can integrate with Git providers, CI/CD systems, and other tools to fully automate project creation.
{{% /alert %}}


## Task {{% param sectionnumber %}}.1: Explore Existing Templates

Before creating your own template, let's explore what's available by default.

1. Navigate to `http://localhost:3000/create`
2. You should see some example templates
3. Click on one to see its parameters and preview

Notice how templates ask for input (like project name, description) and show what will be created.


## Task {{% param sectionnumber %}}.2: Create Your First Template

Let's create a simple template for a Node.js microservice.

Create a new directory for your template:

```bash
mkdir -p ~/backstage-templates/nodejs-microservice
cd ~/backstage-templates/nodejs-microservice
```

Create a `template.yaml` file:

```yaml
apiVersion: scaffolder.backstage.io/v1beta3
kind: Template
metadata:
  name: nodejs-microservice
  title: Node.js Microservice
  description: Create a new Node.js microservice with Express and best practices
  tags:
    - recommended
    - nodejs
    - microservice
spec:
  owner: team-a
  type: service
  
  parameters:
    - title: Service Information
      required:
        - name
        - description
      properties:
        name:
          title: Name
          type: string
          description: Unique name for the service
          ui:autofocus: true
          ui:options:
            rows: 5
        description:
          title: Description
          type: string
          description: A brief description of what this service does
        owner:
          title: Owner
          type: string
          description: Owner of the component
          ui:field: OwnerPicker
          ui:options:
            catalogFilter:
              kind: Group
    
    - title: Repository Information
      required:
        - repoUrl
      properties:
        repoUrl:
          title: Repository Location
          type: string
          ui:field: RepoUrlPicker
          ui:options:
            allowedHosts:
              - github.com

  steps:
    - id: fetch-base
      name: Fetch Base Template
      action: fetch:template
      input:
        url: ./skeleton
        values:
          name: ${{ parameters.name }}
          description: ${{ parameters.description }}
          owner: ${{ parameters.owner }}

    - id: publish
      name: Publish to GitHub
      action: publish:github
      input:
        allowedHosts: ['github.com']
        description: ${{ parameters.description }}
        repoUrl: ${{ parameters.repoUrl }}
        defaultBranch: main

    - id: register
      name: Register Component
      action: catalog:register
      input:
        repoContentsUrl: ${{ steps.publish.output.repoContentsUrl }}
        catalogInfoPath: '/catalog-info.yaml'

  output:
    links:
      - title: Repository
        url: ${{ steps.publish.output.remoteUrl }}
      - title: Open in Catalog
        icon: catalog
        entityRef: ${{ steps.register.output.entityRef }}
```

**Understanding the template structure:**
- `parameters`: Define the input form users fill out
- `steps`: Actions to execute when the template runs
- `output`: Links shown to the user after completion


## Task {{% param sectionnumber %}}.3: Create the Template Skeleton

Templates use a "skeleton" directory containing the actual files to be generated. Let's create one.

Create the skeleton directory structure:

```bash
mkdir -p skeleton/src
```

Create `skeleton/package.json`:

```json
{
  "name": "${{ values.name }}",
  "version": "1.0.0",
  "description": "${{ values.description }}",
  "main": "src/index.js",
  "scripts": {
    "start": "node src/index.js",
    "dev": "nodemon src/index.js",
    "test": "jest"
  },
  "dependencies": {
    "express": "^4.18.2",
    "dotenv": "^16.0.3"
  },
  "devDependencies": {
    "nodemon": "^2.0.22",
    "jest": "^29.5.0"
  }
}
```

Create `skeleton/src/index.js`:

```javascript
const express = require('express');
const app = express();
const PORT = process.env.PORT || 3000;

app.use(express.json());

app.get('/health', (req, res) => {
  res.json({ status: 'healthy', service: '${{ values.name }}' });
});

app.get('/', (req, res) => {
  res.json({ 
    message: 'Welcome to ${{ values.name }}',
    description: '${{ values.description }}'
  });
});

app.listen(PORT, () => {
  console.log(`${{ values.name }} listening on port ${PORT}`);
});

module.exports = app;
```

Create `skeleton/README.md`:

```markdown
# ${{ values.name }}

${{ values.description }}

## Owner

${{ values.owner }}

## Getting Started

### Installation

\`\`\`bash
npm install
\`\`\`

### Running Locally

\`\`\`bash
npm run dev
\`\`\`

### Running Tests

\`\`\`bash
npm test
\`\`\`

## API Endpoints

- `GET /health` - Health check endpoint
- `GET /` - Welcome message
```

Create `skeleton/catalog-info.yaml`:

```yaml
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: ${{ values.name }}
  description: ${{ values.description }}
  annotations:
    github.com/project-slug: ${{ values.repoUrl | parseRepoUrl | pick('owner') }}/${{ values.repoUrl | parseRepoUrl | pick('repo') }}
  tags:
    - nodejs
    - microservice
spec:
  type: service
  lifecycle: experimental
  owner: ${{ values.owner }}
```

Create `skeleton/.gitignore`:

```
node_modules/
.env
*.log
dist/
coverage/
```

{{% alert title="Note" color="primary" %}}
Notice the `${{ values.* }}` syntax - these are template variables that get replaced with user input when the template is executed.
{{% /alert %}}


## Task {{% param sectionnumber %}}.4: Register Your Template

Now let's register this template in Backstage.

Add the template location to your `app-config.yaml`:

```yaml
catalog:
  locations:
    - type: file
      target: ../../backstage-templates/nodejs-microservice/template.yaml
      rules:
        - allow: [Template]
```

Restart your Backstage instance or wait for the catalog to refresh. Navigate to `http://localhost:3000/create` and you should see your new template!


## Task {{% param sectionnumber %}}.5: Use Your Template

Let's create a new service using your template.

1. Go to `http://localhost:3000/create`
2. Click on "Node.js Microservice"
3. Fill in the form:
   - **Name**: `user-service`
   - **Description**: `Service for managing user accounts`
   - **Owner**: Select a team
   - **Repository Location**: Choose GitHub and specify the repository details

4. Click "Review" and then "Create"

Watch as Backstage:
- Creates the repository on GitHub
- Populates it with your template files
- Registers the component in the catalog

{{% alert title="Note" color="primary" %}}
You'll need to have GitHub integration configured with appropriate permissions for this to work. See the previous chapter on catalog setup.
{{% /alert %}}


## Task {{% param sectionnumber %}}.6: Create an Advanced Template with Multiple Steps

Let's create a more sophisticated template that includes CI/CD setup.

Create a new template directory:

```bash
mkdir -p ~/backstage-templates/fullstack-app
cd ~/backstage-templates/fullstack-app
```

Create `template.yaml`:

```yaml
apiVersion: scaffolder.backstage.io/v1beta3
kind: Template
metadata:
  name: fullstack-app
  title: Full-Stack Application
  description: Create a complete full-stack application with React frontend, Node.js backend, and CI/CD
  tags:
    - recommended
    - fullstack
    - react
    - nodejs
spec:
  owner: team-a
  type: website
  
  parameters:
    - title: Application Information
      required:
        - name
        - description
      properties:
        name:
          title: Application Name
          type: string
          description: Unique name for the application
        description:
          title: Description
          type: string
          description: What does this application do?
        owner:
          title: Owner
          type: string
          ui:field: OwnerPicker
          ui:options:
            catalogFilter:
              kind: Group
    
    - title: Technology Choices
      properties:
        database:
          title: Database
          type: string
          description: Which database to use?
          enum:
            - postgresql
            - mysql
            - mongodb
          default: postgresql
        includeAuth:
          title: Include Authentication
          type: boolean
          description: Add authentication scaffolding?
          default: true
    
    - title: Repository Information
      required:
        - repoUrl
      properties:
        repoUrl:
          title: Repository Location
          type: string
          ui:field: RepoUrlPicker
          ui:options:
            allowedHosts:
              - github.com

  steps:
    - id: fetch-base
      name: Fetch Base Template
      action: fetch:template
      input:
        url: ./skeleton
        values:
          name: ${{ parameters.name }}
          description: ${{ parameters.description }}
          owner: ${{ parameters.owner }}
          database: ${{ parameters.database }}
          includeAuth: ${{ parameters.includeAuth }}

    - id: fetch-docs
      name: Fetch Documentation
      action: fetch:plain
      input:
        url: ./docs
        targetPath: ./docs

    - id: publish
      name: Publish to GitHub
      action: publish:github
      input:
        allowedHosts: ['github.com']
        description: ${{ parameters.description }}
        repoUrl: ${{ parameters.repoUrl }}
        defaultBranch: main
        repoVisibility: private
        deleteBranchOnMerge: true
        protectDefaultBranch: false

    - id: create-github-actions
      name: Create GitHub Actions Workflow
      action: fetch:template
      input:
        url: ./workflows
        targetPath: .github/workflows
        values:
          name: ${{ parameters.name }}

    - id: register
      name: Register Component
      action: catalog:register
      input:
        repoContentsUrl: ${{ steps.publish.output.repoContentsUrl }}
        catalogInfoPath: '/catalog-info.yaml'

  output:
    links:
      - title: Repository
        url: ${{ steps.publish.output.remoteUrl }}
      - title: Open in Catalog
        icon: catalog
        entityRef: ${{ steps.register.output.entityRef }}
      - title: CI/CD Pipeline
        url: ${{ steps.publish.output.remoteUrl }}/actions
```

This advanced template demonstrates:
- **Multiple parameter sections**: Organized form with different categories
- **Conditional logic**: Options like `includeAuth` that affect generated code
- **Multiple fetch steps**: Combining different sources
- **CI/CD integration**: Automatically creating GitHub Actions workflows


## Task {{% param sectionnumber %}}.7: Add Custom Template Actions

Backstage allows you to create custom actions for templates. Here's an example of how to add a custom action to send a Slack notification.

In your Backstage backend, you can register custom actions. Create a file `packages/backend/src/plugins/scaffolder.ts`:

```typescript
import { CatalogClient } from '@backstage/catalog-client';
import { createRouter, createBuiltinActions } from '@backstage/plugin-scaffolder-backend';
import { Router } from 'express';
import type { PluginEnvironment } from '../types';
import { ScmIntegrations } from '@backstage/integration';
import { createSlackNotificationAction } from './scaffolder/actions/slack';

export default async function createPlugin(
  env: PluginEnvironment,
): Promise<Router> {
  const catalogClient = new CatalogClient({
    discoveryApi: env.discovery,
  });

  const integrations = ScmIntegrations.fromConfig(env.config);
  const builtInActions = createBuiltinActions({
    integrations,
    catalogClient,
    config: env.config,
    reader: env.reader,
  });

  const actions = [
    ...builtInActions,
    createSlackNotificationAction(),
  ];

  return await createRouter({
    actions,
    logger: env.logger,
    config: env.config,
    database: env.database,
    reader: env.reader,
    catalogClient,
  });
}
```

{{% alert title="Note" color="primary" %}}
Custom actions allow you to integrate templates with any external system - from cloud providers to internal tools.
{{% /alert %}}


## Task {{% param sectionnumber %}}.8: Template Best Practices

As you create templates, follow these best practices:

1. **Start simple**: Begin with basic templates and add complexity gradually
2. **Use clear parameter names**: Make forms intuitive for developers
3. **Provide good defaults**: Reduce the number of required fields
4. **Include documentation**: Add README files and inline comments
5. **Test thoroughly**: Run templates multiple times before sharing
6. **Version your templates**: Use Git tags to version template changes
7. **Collect feedback**: Ask developers what would make templates more useful
8. **Include CI/CD**: Automate testing and deployment from the start
9. **Add validation**: Use JSON Schema to validate user input
10. **Document outputs**: Clearly show what was created and next steps


## Common Template Use Cases

Here are some popular template use cases:

- **Microservices**: Standard service template with logging, metrics, and health checks
- **Frontend applications**: React/Vue/Angular apps with routing and state management
- **Documentation sites**: TechDocs-enabled documentation repositories
- **Infrastructure**: Terraform modules or Kubernetes manifests
- **Libraries**: Shared code libraries with publishing pipelines
- **Data pipelines**: ETL jobs with scheduling and monitoring
- **Mobile apps**: iOS/Android app scaffolding


## Summary

In this chapter, you:
- ✅ Explored existing Backstage templates
- ✅ Created your first software template
- ✅ Built a template skeleton with variables
- ✅ Registered and used your template
- ✅ Created an advanced multi-step template
- ✅ Learned about custom template actions
- ✅ Understood template best practices

Software Templates are one of Backstage's most powerful features for improving developer productivity. By standardizing project creation, you reduce cognitive load and ensure consistency across your organization.  
