# DevSecOps Pipeline for Vulnado

This document describes the DevSecOps pipeline implementation for the Vulnado project using Azure DevOps. The pipeline integrates security at various stages of the development lifecycle, including Static Application Security Testing (SAST), Software Composition Analysis (SCA), and Dynamic Application Security Testing (DAST).

## Pipeline Overview

The DevSecOps pipeline consists of the following stages:

1. **Build and Unit Tests**: Compiles the application and runs unit tests
2. **SAST with SonarQube (No Code Coverage)**: Performs static code analysis without code coverage metrics
3. **SAST with SonarQube (With Code Coverage)**: Performs static code analysis with JaCoCo code coverage metrics
4. **SCA with Snyk**: Analyzes dependencies for known vulnerabilities
5. **DAST with OWASP ZAP**: Performs dynamic security testing on the running application

## Setup Instructions

### Prerequisites

- Azure DevOps account with appropriate permissions
- SonarCloud account and organization
- Snyk account and organization
- Maven installed locally for development testing

### Configuration

#### Azure DevOps Service Connections

1. **SonarCloud Connection**:
   - Go to Project Settings > Service Connections > New Service Connection > SonarCloud
   - Enter your SonarCloud token and name the connection "SonarCloud"

2. **Snyk Connection**:
   - Go to Project Settings > Service Connections > New Service Connection > Snyk
   - Enter your Snyk API token and name the connection "Snyk"

#### Environment Variables

The pipeline uses the following environment variables that should be configured in Azure DevOps:

- `SONAR_ORGANIZATION`: Your SonarCloud organization name
- `SONAR_PROJECT_KEY`: The project key in SonarCloud (default: vulnado)
- `SNYK_ORGANIZATION`: Your Snyk organization name
- `SNYK_API_TOKEN`: Your Snyk API token (should be stored as a secret variable)
- `ZAP_TARGET_URL`: The URL of the deployed application for DAST testing

## Component Details

### 1. SAST with SonarQube

#### Without Code Coverage

This stage performs static code analysis to identify code quality issues and security vulnerabilities without measuring code coverage.

Key configurations in `pom.xml`:
```xml
<properties>
    <sonar.projectKey>vulnado</sonar.projectKey>
    <sonar.organization>your-sonar-organization</sonar.organization>
    <sonar.host.url>https://sonarcloud.io</sonar.host.url>
</properties>
```

#### With Code Coverage (JaCoCo)

This stage includes code coverage metrics using JaCoCo, which helps identify untested code that might contain vulnerabilities.

Key configurations in `pom.xml`:
```xml
<plugin>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <version>0.8.7</version>
    <executions>
        <execution>
            <id>prepare-agent</id>
            <goals>
                <goal>prepare-agent</goal>
            </goals>
        </execution>
        <execution>
            <id>report</id>
            <phase>verify</phase>
            <goals>
                <goal>report</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

### 2. SCA with Snyk

This stage scans dependencies for known vulnerabilities using Snyk.

Key configurations in `pom.xml`:
```xml
<plugin>
    <groupId>io.snyk</groupId>
    <artifactId>snyk-maven-plugin</artifactId>
    <version>2.1.0</version>
    <executions>
        <execution>
            <id>snyk-test</id>
            <phase>test</phase>
            <goals>
                <goal>test</goal>
            </goals>
        </execution>
        <execution>
            <id>snyk-monitor</id>
            <phase>install</phase>
            <goals>
                <goal>monitor</goal>
            </goals>
        </execution>
    </executions>
    <configuration>
        <apiToken>${env.SNYK_API_TOKEN}</apiToken>
        <failOnIssues>false</failOnIssues>
        <org>${env.SNYK_ORGANIZATION}</org>
    </configuration>
</plugin>
```

### 3. DAST with OWASP ZAP

This stage performs dynamic security testing on the running application using OWASP ZAP.

The pipeline deploys the application temporarily for testing, then runs ZAP against it to identify runtime vulnerabilities.

## Interpreting Results

### SonarQube Results

1. Access your SonarCloud dashboard to view the analysis results
2. Focus on:
   - Security Hotspots: Code that might be vulnerable
   - Vulnerabilities: Confirmed security issues
   - Code Smells: Maintainability issues
   - Coverage: Percentage of code covered by tests

### Snyk Results

1. Access your Snyk dashboard to view dependency vulnerabilities
2. For each vulnerability, consider:
   - Severity (Critical, High, Medium, Low)
   - Whether it affects your application
   - Available remediation options (upgrade, patch, ignore)

### OWASP ZAP Results

1. Review the ZAP scan report in Azure DevOps
2. Focus on:
   - High-risk vulnerabilities (e.g., SQL Injection, XSS)
   - Authentication issues
   - Information disclosure

## Dependency Management

As part of implementing this DevSecOps pipeline, we've updated several dependencies to address security vulnerabilities:

1. **Spring Boot**: Updated from 2.1.2.RELEASE to 2.7.14
   - Addresses critical vulnerabilities in Spring Boot and related components

2. **PostgreSQL**: Updated from 42.2.5 to 42.7.1
   - Addresses 6 known vulnerabilities with CRITICAL severity
   - Note: Even the latest version (42.7.1) may have a recently discovered vulnerability, demonstrating the need for continuous security monitoring

3. **JSoup**: Updated from 1.8.3 to 1.15.4
   - Addresses 2 known vulnerabilities with HIGH severity

These updates demonstrate the value of the SCA scanning with Snyk, which will automatically detect such vulnerabilities in your dependencies.

## Best Practices

1. **Fix Security Issues Early**: Address security findings as soon as they are identified
2. **Prioritize Vulnerabilities**: Focus on high-severity issues first
3. **Increase Test Coverage**: Aim for at least 80% code coverage
4. **Regular Scanning**: Run the pipeline regularly, not just before releases
5. **Security Training**: Ensure developers understand common vulnerabilities and how to prevent them
6. **Keep Dependencies Updated**: Regularly update dependencies to their latest secure versions
7. **Use Dependency Locking**: Consider using dependency locking to ensure consistent builds
8. **Review Security Bulletins**: Subscribe to security bulletins for your key dependencies

## Customization

The pipeline can be customized by:

1. Modifying the `azure-pipelines.yml` file to add or remove stages
2. Updating the plugin configurations in `pom.xml`
3. Adding additional security tools as needed

## Troubleshooting

### Common Issues

1. **SonarQube Analysis Fails**:
   - Verify SonarCloud connection settings
   - Ensure the project exists in SonarCloud
   - Check that the organization name is correct

2. **Snyk Scan Fails**:
   - Verify Snyk API token
   - Check organization name
   - Ensure network connectivity to Snyk API

3. **OWASP ZAP Scan Fails**:
   - Verify the application is running and accessible
   - Check the target URL configuration
   - Increase the timeout for the ZAP scan if needed

## References

- [Azure DevOps Documentation](https://docs.microsoft.com/en-us/azure/devops/)
- [SonarQube Documentation](https://docs.sonarqube.org/)
- [JaCoCo Documentation](https://www.jacoco.org/jacoco/trunk/doc/)
- [Snyk Documentation](https://docs.snyk.io/)
- [OWASP ZAP Documentation](https://www.zaproxy.org/docs/)