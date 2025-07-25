# E-Commerce Platform CI/CD Pipeline

## Project Overview

This project demonstrates the implementation of a complete CI/CD pipeline for an e-commerce platform using GitHub Actions. The platform consists of two main components: a backend API service and a frontend web application, both containerized with Docker and deployed to the cloud with automated workflows.

## Architecture

```
ecommerce-platform/
├── api/                    # Backend Node.js/Express API
│   ├── src/
│   ├── tests/
│   ├── package.json
│   └── Dockerfile
├── webapp/                 # Frontend React Application
│   ├── src/
│   ├── public/
│   ├── package.json
│   └── Dockerfile
├── .github/
│   └── workflows/
│       ├── backend-ci-cd.yml
│       └── frontend-ci-cd.yml
└── README.md
```

## Prerequisites

- Node.js (v16 or higher)
- Docker and Docker Compose
- GitHub account
- Cloud platform account (AWS/Azure/GCP)
- Git

## Implementation Steps

### Task 1: Project Setup

**Step 1.1: Create GitHub Repository**
```bash
# Create new repository on GitHub named "ecommerce-platform"
git clone https://github.com/yourusername/ecommerce-platform.git
cd ecommerce-platform
```

**Step 1.2: Create Directory Structure**
```bash
mkdir api webapp
mkdir -p .github/workflows
```

### Task 2: Initialize GitHub Actions

**Step 2.1: Initialize Git Repository**
```bash
git init
git add .
git commit -m "Initial project structure"
git push origin main
```

**Step 2.2: Create Workflows Directory**
```bash
mkdir -p .github/workflows
```

### Task 3: Backend API Setup

**Step 3.1: Initialize Node.js Application**
```bash
cd api
npm init -y
npm install express cors helmet morgan dotenv bcryptjs jsonwebtoken
npm install --save-dev jest supertest nodemon
```

**Step 3.2: Create Basic API Structure**
```javascript
// api/src/app.js
const express = require('express');
const cors = require('cors');
const helmet = require('helmet');
const app = express();

app.use(helmet());
app.use(cors());
app.use(express.json());

// Routes
app.get('/api/health', (req, res) => {
  res.json({ status: 'OK', message: 'API is running' });
});

app.get('/api/products', (req, res) => {
  res.json([
    { id: 1, name: 'Product 1', price: 29.99 },
    { id: 2, name: 'Product 2', price: 39.99 }
  ]);
});

module.exports = app;
```

**Step 3.3: Implement Unit Tests**
```javascript
// api/tests/app.test.js
const request = require('supertest');
const app = require('../src/app');

describe('API Endpoints', () => {
  test('GET /api/health should return OK', async () => {
    const response = await request(app).get('/api/health');
    expect(response.status).toBe(200);
    expect(response.body.status).toBe('OK');
  });

  test('GET /api/products should return products array', async () => {
    const response = await request(app).get('/api/products');
    expect(response.status).toBe(200);
    expect(Array.isArray(response.body)).toBe(true);
  });
});
```

### Task 4: Frontend Web Application Setup

**Step 4.1: Create React Application**
```bash
cd webapp
npx create-react-app . --template typescript
npm install axios react-router-dom
```

**Step 4.2: Create Basic Components**
```jsx
// webapp/src/components/ProductList.jsx
import React, { useState, useEffect } from 'react';
import axios from 'axios';

const ProductList = () => {
  const [products, setProducts] = useState([]);

  useEffect(() => {
    axios.get('/api/products')
      .then(response => setProducts(response.data))
      .catch(error => console.error('Error fetching products:', error));
  }, []);

  return (
    <div>
      <h2>Products</h2>
      {products.map(product => (
        <div key={product.id}>
          <h3>{product.name}</h3>
          <p>${product.price}</p>
        </div>
      ))}
    </div>
  );
};

export default ProductList;
```

### Task 5: Continuous Integration Workflow

**Step 5.1: Backend CI/CD Workflow**
```yaml
# .github/workflows/backend-ci-cd.yml
name: Backend CI/CD

on:
  push:
    branches: [main]
    paths: ['api/**']
  pull_request:
    branches: [main]
    paths: ['api/**']

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
          cache-dependency-path: api/package-lock.json
      
      - name: Install dependencies
        run: |
          cd api
          npm ci
      
      - name: Run tests
        run: |
          cd api
          npm test
      
      - name: Build application
        run: |
          cd api
          npm run build
```

**Step 5.2: Frontend CI/CD Workflow**
```yaml
# .github/workflows/frontend-ci-cd.yml
name: Frontend CI/CD

on:
  push:
    branches: [main]
    paths: ['webapp/**']
  pull_request:
    branches: [main]
    paths: ['webapp/**']

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
          cache-dependency-path: webapp/package-lock.json
      
      - name: Install dependencies
        run: |
          cd webapp
          npm ci
      
      - name: Run tests
        run: |
          cd webapp
          npm test -- --coverage --watchAll=false
      
      - name: Build application
        run: |
          cd webapp
          npm run build
```

### Task 6: Docker Integration

**Step 6.1: Backend Dockerfile**
```dockerfile
# api/Dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

COPY . .

EXPOSE 3000

CMD ["npm", "start"]
```

**Step 6.2: Frontend Dockerfile**
```dockerfile
# webapp/Dockerfile
FROM node:18-alpine as build

WORKDIR /app
COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html
COPY nginx.conf /etc/nginx/nginx.conf

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

**Step 6.3: Update Workflows for Docker**
```yaml
# Add to both workflow files
  build-docker:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Build Docker image
        run: |
          docker build -t ecommerce-api:${{ github.sha }} ./api
      
      - name: Login to Docker Hub
        if: github.ref == 'refs/heads/main'
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}
      
      - name: Push Docker image
        if: github.ref == 'refs/heads/main'
        run: |
          docker tag ecommerce-api:${{ github.sha }} your-dockerhub/ecommerce-api:latest
          docker push your-dockerhub/ecommerce-api:latest
```

### Task 7: Deploy to Cloud

**Step 7.1: AWS Deployment Configuration**
```yaml
# Add to workflow files
  deploy:
    needs: build-docker
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1
      
      - name: Deploy to ECS
        run: |
          aws ecs update-service \
            --cluster ecommerce-cluster \
            --service ecommerce-api \
            --force-new-deployment
```

**Step 7.2: Required GitHub Secrets**
```
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
DOCKER_USERNAME
DOCKER_PASSWORD
DATABASE_URL
JWT_SECRET
```

### Task 8: Continuous Deployment

**Step 8.1: Automatic Deployment on Main Branch**
The workflows are configured to automatically deploy when changes are pushed to the main branch, ensuring continuous deployment.

**Step 8.2: Environment-based Deployment**
```yaml
# Add environment-specific deployments
  deploy-staging:
    if: github.ref == 'refs/heads/develop'
    # Staging deployment steps
  
  deploy-production:
    if: github.ref == 'refs/heads/main'
    # Production deployment steps
```

### Task 9: Performance and Security

**Step 9.1: Implement Caching**
```yaml
# Already implemented in workflows above
- name: Setup Node.js
  uses: actions/setup-node@v3
  with:
    node-version: '18'
    cache: 'npm'
    cache-dependency-path: api/package-lock.json
```

**Step 9.2: Security Best Practices**
- All sensitive data stored as GitHub Secrets
- Docker images scanned for vulnerabilities
- Dependencies regularly updated
- HTTPS enforced for all communications

### Task 10: Project Documentation

This README.md file serves as the comprehensive documentation for the project setup, workflow details, and local development instructions.

## Local Development

### Prerequisites Installation
```bash
# Install Node.js dependencies
cd api && npm install
cd ../webapp && npm install
```

### Running the Application
```bash
# Terminal 1: Start backend
cd api
npm run dev

# Terminal 2: Start frontend
cd webapp
npm start
```

### Running Tests
```bash
# Backend tests
cd api && npm test

# Frontend tests
cd webapp && npm test
```

### Docker Development
```bash
# Build and run with Docker Compose
docker-compose up --build
```

## Environment Variables

### Backend (.env)
```
NODE_ENV=development
PORT=3000
DATABASE_URL=your_database_url
JWT_SECRET=your_jwt_secret
```

### Frontend (.env)
```
REACT_APP_API_URL=http://localhost:3000
```

## Deployment

The application is automatically deployed when changes are pushed to the main branch. The CI/CD pipeline handles:

1. **Testing**: Runs unit tests for both frontend and backend
2. **Building**: Creates optimized production builds
3. **Containerization**: Builds Docker images
4. **Deployment**: Deploys to cloud infrastructure
5. **Monitoring**: Tracks deployment status and application health

## Troubleshooting

### Common Issues

1. **Build Failures**: Check Node.js version compatibility
2. **Test Failures**: Ensure all dependencies are installed
3. **Docker Issues**: Verify Docker daemon is running
4. **Deployment Failures**: Check cloud credentials and permissions

### Support

For issues and questions:
1. Check the [Issues](https://github.com/yourusername/ecommerce-platform/issues) page
2. Review the workflow logs in GitHub Actions
3. Contact the development team

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.