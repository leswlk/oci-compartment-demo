# OCI Compartment Demo

**A hands-on demonstration of secure and organized OCI environment design, focusing on compartments, IAM policies, and tagging for cost tracking.**

[![License: MIT](https://img.shields.io/github/license/leswlk/oci-compartment-demo.svg)](https://github.com/leswlk/oci-compartment-demo)

## Overview

This project demonstrates the implementation of a secure and well-organized OCI (Oracle Cloud Infrastructure) environment. It focuses on leveraging key OCI services to isolate resources (using compartments), control access (with IAM policies and dynamic groups), and track costs (using tags). This project is designed to be a practical learning exercise and a sample configuration for a public sector organization.

## Objectives

*   **Compartmentalization:** Demonstrate the creation and organization of resources into distinct compartments for logical separation.
*   **IAM Security:** Implement compartment-level IAM policies for least privilege access control.
*   **Dynamic Group Management:** Utilize dynamic groups based on attributes (e.g., department, role) for streamlined user access management.
*   **Cost Tracking:** Apply tags to resources for accurate cost analysis and reporting.

## Architecture

The project utilizes a three-compartment structure:

*   **Health:**  Resources related to healthcare data and applications.
*   **Education:** Resources for educational applications and data.
*   **Infrastructure:** Core infrastructure components (e.g., networking, storage).

[You can include a link to a text-based architecture diagram here if you created one.]

## Getting Started

1.  **Prerequisites:**
    *   An active Oracle Cloud Infrastructure account.
    *   Basic familiarity with the OCI console.

2.  **Setup:**
    *   Follow the steps outlined in the "Steps" section of the README.md (this is where you would paste the detailed steps).

## Future Enhancements

*   Implement budgets and alerting.
*   Automate deployment using Infrastructure as Code (IaC).
*   Add more detailed documentation and diagrams.

## License

This project is licensed under the MIT License – see the [LICENSE](LICENSE) file for details.

