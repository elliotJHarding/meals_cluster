# Meals Platform - ArgoCD GitOps Repository

This repository contains the Kubernetes manifests for the Meals Platform managed via ArgoCD GitOps.

## Architecture

The platform consists of:
- **meals**: Backend API service (Java/Spring Boot)
- **meals-web-client**: Frontend React application  
- **meals-opengraph**: SEO/OpenGraph service for social media previews
- **Infrastructure**: Ingress routing with TLS termination

## Directory Structure

```
├── argocd-applications/          # ArgoCD Application definitions
│   ├── root-app.yaml            # App-of-apps root application
│   ├── meals-app.yaml           # Backend service application
│   ├── meals-web-client-app.yaml # Frontend application
│   ├── meals-opengraph-app.yaml # OpenGraph service application
│   └── meals-infrastructure-app.yaml # Infrastructure application
├── apps/                        # Application-specific manifests
│   ├── meals/                   # Backend service
│   ├── meals-web-client/        # Frontend 
│   └── meals-opengraph/         # OpenGraph service
├── infrastructure/              # Infrastructure components
│   └── ingress/                # Ingress configuration
└── k8s-cluster-config/         # Original exported cluster state (reference)
```

## Getting Started

### Prerequisites
- ArgoCD installed on your cluster
- Repository access configured in ArgoCD

### Setup Instructions

1. **Update Repository URLs**: 
   Edit all ArgoCD Application files to replace `https://github.com/your-username/meals_cluster.git` with your actual repository URL.

2. **Deploy Secrets** (⚠️ **Critical - Do this first**):
   ```bash
   # Create your production secrets file
   cp secrets/meals-secrets.template.yaml secrets/meals-secrets.yaml
   # Edit secrets/meals-secrets.yaml with your actual base64-encoded values
   
   # Deploy secrets to cluster
   ./deploy-secrets.sh
   ```

3. **Deploy the Root Application**:
   ```bash
   kubectl apply -f argocd-applications/root-app.yaml
   ```

4. **Verify Deployment**:
   - Check ArgoCD UI for application status
   - Verify all applications are synced and healthy
   - Ensure pods can access secrets without errors

### Configuration

#### Environment Variables & Secrets
The meals backend configuration is split between environment variables and Kubernetes secrets:

**Environment Variables** (non-sensitive):
- Database/Redis URLs and connection parameters
- Application URLs and redirect URLs
- Feature flags and configuration options

**Kubernetes Secrets** (sensitive data):
- Database username and password
- Redis password
- Google OAuth client secret
- JWT/session token secrets

🔐 **Security Implementation**: All sensitive values are stored as Kubernetes secrets and referenced via `secretKeyRef`. See the [Secrets Management Guide](secrets/README.md) for details.

#### Domain Configuration
- **Primary Domain**: `grubplanner.co.uk`
- **TLS**: Managed by cert-manager with Let's Encrypt
- **Routing**: 
  - `/` → meals-web-client (frontend)
  - `/api/` → meals (backend API)
  - `/opengraph/` → meals-opengraph (SEO service)

### Management

#### Updating Applications
1. Update the relevant manifests in `apps/` directories
2. Commit and push changes
3. ArgoCD will automatically sync (if auto-sync is enabled)

#### Adding New Applications
1. Create new directory under `apps/`
2. Add Kubernetes manifests and kustomization.yaml
3. Create corresponding ArgoCD Application in `argocd-applications/`
4. Commit changes - root app will deploy the new application

#### Rollbacks
Use ArgoCD UI or CLI to rollback to previous revisions:
```bash
argocd app rollback meals-platform --revision 1
```

### Excluded Resources

The following auto-generated resources are intentionally excluded from GitOps management:
- cert-manager controllers and webhooks
- Calico CNI components  
- CoreDNS, metrics-server, dashboard
- microk8s system components
- Auto-provisioned PVs/PVCs
- kube-root-ca.crt ConfigMaps

These are managed by the underlying platform (microk8s) and should not be version controlled.

## Troubleshooting

### Common Issues

1. **Applications Stuck in Progressing**: Check resource quotas and node capacity
2. **Image Pull Errors**: Verify registry access and image tags
3. **Service Connectivity**: Check service names match in ingress configuration
4. **TLS Certificate Issues**: Verify cert-manager issuer configuration

### Monitoring

- **ArgoCD UI**: Monitor application sync status and health
- **Kubernetes Dashboard**: View resource status and logs
- **Application Logs**: Use `kubectl logs` for debugging

For more detailed troubleshooting, check the ArgoCD documentation and application-specific logs.

## Security

### Secrets Management
This repository implements secure secret management:

- **🔐 Kubernetes Secrets**: All sensitive data stored as secrets
- **🚫 No Plain Text**: No sensitive values in deployment manifests
- **📁 Separated Storage**: Actual secrets kept separate from application code
- **🔄 Template System**: Safe templates for version control
- **🛡️ Git Protection**: .gitignore prevents accidental secret commits

### Security Best Practices Implemented
- ✅ Database credentials secured
- ✅ API keys and tokens secured  
- ✅ OAuth secrets secured
- ✅ Separation of sensitive/non-sensitive config
- ✅ Base64 encoding for Kubernetes compliance
- ✅ Comprehensive deployment automation
- ✅ Documentation and guidelines

For detailed security procedures, see [Secrets Management Guide](secrets/README.md).