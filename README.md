## UdaPeople — Auto-Deploy CI/CD Pipeline

This repository is a fork of the [Udacity cdond-c3-projectstarter](https://github.com/udacity/cdond-c3-projectstarter). The upstream repo provided the application source (a NestJS backend and React/Redux frontend) along with CloudFormation templates and empty Ansible skeleton files. Everything described below was implemented on top of that starting point.

![Diagram of CI/CD Pipeline](udapeople.png)

### What Was Extended

#### CircleCI Pipeline (`.circleci/config.yml`)

The pipeline was built from scratch. The upstream repo shipped only a placeholder config. The implemented workflow covers:

- **Build & test jobs** for both frontend and backend, with dependency caching between jobs.
- **Security scan jobs** (`npm audit`) for frontend and backend, including forced fixes for critical vulnerabilities.
- **Infrastructure provisioning** (`deploy-infrastructure`) using CloudFormation to create the EC2 backend instance and S3 frontend bucket, writing the new EC2 IP into an Ansible inventory file persisted across jobs via CircleCI workspaces.
- **Server configuration** (`configure-infrastructure`) via Ansible, installing Node.js, environment variables, and Prometheus node exporter on the EC2 instance.
- **Database migrations** (`run-migrations`) running TypeORM migrations against the provisioned RDS-compatible PostgreSQL instance, with the migration result stored in kvdb.io so rollback commands can query it.
- **Frontend deployment** (`deploy-frontend`) injecting the backend EC2 IP as `API_URL` at build time, then uploading the compiled `dist/` to the S3 bucket.
- **Backend deployment** (`deploy-backend`) compiling the NestJS app, compressing it into a tarball, and deploying it to EC2 via Ansible.
- **Smoke tests** hitting `GET /api/status` on the backend and the S3 static URL on the frontend to verify the deployment is live.
- **Blue/green promotion** (`cloudfront-update`) swapping the CloudFront origin to the newly deployed S3 bucket.
- **Cleanup** (`cleanup`) deleting the previous environment’s CloudFormation stacks and S3 bucket after a successful promotion, with the old workflow ID persisted in `.circleci/OldWorkflowID.txt`.

Deployment jobs are gated to run only on pushes to `master`. Every deployment job attaches `destroy_environment` and `revert_migrations` handlers that fire `on_fail` to ensure automatic rollback on any failure.

#### Ansible (`configure-server.yml`, `deploy-backend.yml`)

The Ansible playbooks were written to install all required system dependencies on the EC2 instance, configure environment variables, set up and start the backend application process, and install and register the Prometheus node exporter as a service.

#### Dependency Fixes

Both the frontend and backend had known vulnerabilities in their dependency trees. `npm audit fix` was applied and the `oauth-sign` package was pinned in the frontend to resolve critical audit findings.

### Built With

- [Circle CI](www.circleci.com) - Cloud-based CI/CD service
- [Amazon AWS](https://aws.amazon.com/) - Cloud services
- [AWS CLI](https://aws.amazon.com/cli/) - Command-line tool for AWS
- [CloudFormation](https://aws.amazon.com/cloudformation/) - Infrastrcuture as code
- [Ansible](https://www.ansible.com/) - Configuration management tool
- [Prometheus](https://prometheus.io/) - Monitoring tool

### License

[License](LICENSE.md)
