# Helm Charts for CodeDesignPlus Microservices

This repository contains Helm charts for deploying microservices developed with the CodeDesignPlus SDK on Kubernetes. Each chart is designed to provide best practices for deployment, configuration, scaling, and integration with tools like Istio and Vault.

## Artifacts Hub

The Artifacts Hub provides a centralized location for all Helm chart artifacts, including pre-built images, configuration files, and other dependencies required for deploying CodeDesignPlus microservices.

[![Artifact Hub](https://img.shields.io/endpoint?url=https://artifacthub.io/badge/repository/codedesignplus-charts)](https://artifacthub.io/packages/search?repo=codedesignplus-charts)

## Charts Overview


| Chart Name                | Description                                                                                  |
|--------------------------|----------------------------------------------------------------------------------------------|
| [ms-base](./charts/ms-base/Readme.md)                  | Base chart providing common configuration, deployment templates, Vault, Istio, autoscaling, and best practices for all microservices. |
| [ms-catalogs-rest](./charts/ms-catalogs-rest/Readme.md)         | REST API microservice for managing catalogs. Inherits from ms-base.                          |
| [ms-emails-grpc](./charts/ms-emails-grpc/Readme.md)           | gRPC microservice for email delivery and notifications. Inherits from ms-base.               |
| [ms-emails-rest](./charts/ms-emails-rest/Readme.md)           | REST API microservice for email delivery and notifications. Inherits from ms-base.           |
| [ms-emails-worker](./charts/ms-emails-worker/Readme.md)         | Worker microservice for background email processing. Inherits from ms-base.                  |
| [ms-filestorage-rest](./charts/ms-filestorage-rest/Readme.md)      | REST API microservice for file storage operations. Inherits from ms-base.                    |
| [ms-filestorage-worker](./charts/ms-filestorage-worker/Readme.md)    | Worker microservice for background file storage tasks. Inherits from ms-base.                |
| [ms-licenses-rest](./charts/ms-licenses-rest/Readme.md)         | REST API microservice for license management. Inherits from ms-base.                         |
| [ms-locations-rest](./charts/ms-locations-rest/Readme.md)        | REST API microservice for location management. Inherits from ms-base.                        |
| [ms-microsoftgraph-rest](./charts/ms-microsoftgraph-rest/Readme.md)   | REST API microservice for Microsoft Graph integrations. Inherits from ms-base.               |
| [ms-microsoftgraph-worker](./charts/ms-microsoftgraph-worker/Readme.md) | Worker microservice for background Microsoft Graph tasks. Inherits from ms-base.             |
| [ms-modules-rest](./charts/ms-modules-rest/Readme.md)          | REST API microservice for module management. Inherits from ms-base.                          |
| [ms-payments-grpc](./charts/ms-payments-grpc/Readme.md)         | gRPC microservice for payment processing. Inherits from ms-base.                             |
| [ms-payments-rest](./charts/ms-payments-rest/Readme.md)         | REST API microservice for payment processing. Inherits from ms-base.                         |
| [ms-payments-worker](./charts/ms-payments-worker/Readme.md)       | Worker microservice for background payment tasks. Inherits from ms-base.                     |
| [ms-rbac-grpc](./charts/ms-rbac-grpc/Readme.md)             | gRPC microservice for role-based access control. Inherits from ms-base.                      |
| [ms-rbac-rest](./charts/ms-rbac-rest/Readme.md)             | REST API microservice for role-based access control. Inherits from ms-base.                  |
| [ms-roles-rest](./charts/ms-roles-rest/Readme.md)            | REST API microservice for role management. Inherits from ms-base.                            |
| [ms-services-grpc](./charts/ms-services-grpc/Readme.md)         | gRPC microservice for service orchestration. Inherits from ms-base.                          |
| [ms-services-rest](./charts/ms-services-rest/Readme.md)         | REST API microservice for service orchestration. Inherits from ms-base.                      |
| [ms-tenants-grpc](./charts/ms-tenants-grpc/Readme.md)          | gRPC microservice for tenant management. Inherits from ms-base.                              |
| [ms-tenants-rest](./charts/ms-tenants-rest/Readme.md)          | REST API microservice for tenant management. Inherits from ms-base.                          |
| [ms-users-grpc](./charts/ms-users-grpc/Readme.md)            | gRPC microservice for user management. Inherits from ms-base.                                |
| [ms-users-rest](./charts/ms-users-rest/Readme.md)            | REST API microservice for user management. Inherits from ms-base.                            |
| [ms-users-worker](./charts/ms-users-worker/Readme.md)          | Worker microservice for background user management tasks. Inherits from ms-base.             |

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
