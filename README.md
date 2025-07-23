# Helm Charts for CodeDesignPlus Microservices

This repository contains Helm charts for deploying microservices developed with the CodeDesignPlus SDK on Kubernetes. Each chart is designed to provide best practices for deployment, configuration, scaling, and integration with tools like Istio and Vault.


## Charts Overview

## Charts Overview

Below is a description of each chart included in this repository:

- **ms-base**: Base chart providing common configuration, deployment templates, Vault, Istio, autoscaling, and best practices for all microservices.
- **ms-catalogs-rest**: REST API microservice for managing catalogs. Inherits from ms-base.
- **ms-emails-grpc**: gRPC microservice for email delivery and notifications. Inherits from ms-base.
- **ms-emails-rest**: REST API microservice for email delivery and notifications. Inherits from ms-base.
- **ms-emails-worker**: Worker microservice for background email processing. Inherits from ms-base.
- **ms-filestorage-rest**: REST API microservice for file storage operations. Inherits from ms-base.
- **ms-filestorage-worker**: Worker microservice for background file storage tasks. Inherits from ms-base.
- **ms-licenses-rest**: REST API microservice for license management. Inherits from ms-base.
- **ms-locations-rest**: REST API microservice for location management. Inherits from ms-base.
- **ms-microsoftgraph-rest**: REST API microservice for Microsoft Graph integrations. Inherits from ms-base.
- **ms-microsoftgraph-worker**: Worker microservice for background Microsoft Graph tasks. Inherits from ms-base.
- **ms-modules-rest**: REST API microservice for module management. Inherits from ms-base.
- **ms-payments-grpc**: gRPC microservice for payment processing. Inherits from ms-base.
- **ms-payments-rest**: REST API microservice for payment processing. Inherits from ms-base.
- **ms-payments-worker**: Worker microservice for background payment tasks. Inherits from ms-base.
- **ms-rbac-grpc**: gRPC microservice for role-based access control. Inherits from ms-base.
- **ms-rbac-rest**: REST API microservice for role-based access control. Inherits from ms-base.
- **ms-roles-rest**: REST API microservice for role management. Inherits from ms-base.
- **ms-services-grpc**: gRPC microservice for service orchestration. Inherits from ms-base.
- **ms-services-rest**: REST API microservice for service orchestration. Inherits from ms-base.
- **ms-tenants-grpc**: gRPC microservice for tenant management. Inherits from ms-base.
- **ms-tenants-rest**: REST API microservice for tenant management. Inherits from ms-base.
- **ms-users-grpc**: gRPC microservice for user management. Inherits from ms-base.
- **ms-users-rest**: REST API microservice for user management. Inherits from ms-base.
- **ms-users-worker**: Worker microservice for background user management tasks. Inherits from ms-base.

Each chart includes its own README with usage instructions and configurable values.

## Usage

1. Clone this repository:
   ```sh
   git clone https://github.com/codedesignplus/helm-charts.git
   cd helm-charts
   ```
2. Review and customize the values in each chart's `values.yaml` as needed.
3. Deploy a microservice using Helm:
   ```sh
   helm upgrade --install <release-name> codedesignplus/<chart-name> \
       --namespace <namespace> \
       --set ms-base.vault.token=<vault-token> \
       --create-namespace
   ```
4. For advanced configuration, refer to each chart's README and the `ms-base` chart documentation.

## Requirements

- Kubernetes >= 1.20
- Helm >= 3.0
- Istio (optional, for VirtualService integration)
- Vault (optional, for secret management)

## Contributing

Contributions are welcome! Please open issues or submit pull requests for improvements, new charts, or bug fixes.

## License

This project is licensed under the LGPL-3.0 License - see the [LICENSE](LICENSE.md) file for details.
