# Example Git Flow Standard

## Table of Contents
- [Branch Structure](#branch-structure)
- [Workflow](#workflow)
  - [1. Work Package and Feature Development](#1-work-package-and-feature-development)
  - [2. Internal QA and Testing](#2-internal-qa-and-testing)
  - [3. Bug Fixes During Development and In-house QA](#3-bug-fixes-during-development-and-in-house-qa)
  - [4. Release Preparation and On-site Testing](#4-release-preparation-and-on-site-testing)
  - [5. On-site Testing and Go-Live](#5-on-site-testing-and-go-live)
  - [6. Hotfixes](#6-hotfixes)
  - [7. System Acceptance and Support](#7-system-acceptance-and-support)
  - [8. New Feature Development for Subsequent Releases](#8-new-feature-development-for-subsequent-releases)
- [Best Practices](#best-practices)
- [Handling Dependencies Between Work Packages](#handling-dependencies-between-work-packages)
  - [Dependency Management Process](#dependency-management-process)
  - [Best Practices for Managing Dependencies](#best-practices-for-managing-dependencies)

## Branch Structure
```mermaid
gitGraph
    commit
    branch feature/wp1/feature-a
    commit
    commit
    checkout main
    branch feature/wp2/feature-b
    commit
    checkout main
    branch release/1.0
    commit
    checkout main
    branch hotfix/critical-bug
    commit
    checkout main
    merge hotfix/critical-bug
    checkout release/1.0
    merge feature/wp1/feature-a
    merge feature/wp2/feature-b
    checkout main
    merge release/1.0
    branch support/1.0
    commit
```
| Branch Type | Naming Convention | Purpose |
|-------------|-------------------|---------|
| main | `main` | The stable, production-ready branch |
| Feature | `feature/<work-package>/<feature-name>` | Development branches for each work package or feature |
| Release | `release/<version>` | Temporary branches for release candidates |
| Hotfix | `hotfix/<fix-name>` | Urgent fixes for production issues |
| Support | `support/<version>` | Long-term support for accepted systems |

## Workflow

```mermaid
graph TD
    A[Feature Development] --> B[Internal QA and Testing]
    B --> C{Bugs Found?}
    C -->|Yes| D[Bug Fixes]
    D --> A
    C -->|No| E[Release Preparation]
    E --> F[On-site Testing]
    F --> G{Issues Found?}
    G -->|Yes| H[Hotfixes]
    H --> F
    G -->|No| I[Go-Live]
    I --> J[System Acceptance]
    J --> K[Long-term Support]
    K --> L[New Feature Development]
    L --> A
```

### 1. Work Package and Feature Development

1. Create feature branches from `main`:
   ```bash
   git checkout -b feature/wp1/feature-a main
   ```
2. Develop and test features within these branches
3. Use pull requests for code review before merging

### 2. Internal QA and Testing

- Conduct testing on feature branches
- Create bugfix branches when needed:
  ```bash
  git checkout -b bugfix/fix-description feature/wp1/feature-a
  ```
- Merge bugfix branches back into their respective feature branches
### 3. Bug Fixes During Development and In-house QA

During the development phase and in-house QA, bugs will inevitably be discovered. Here's how to handle them:

```mermaid
flowchart TD
    A[Bug Discovered] --> B{Minor or Major?}
    B -->|Minor| C[Fix in Feature Branch]
    B -->|Major| D[Create Bug Fix Branch]
    C --> E[Commit Changes]
    D --> F[Implement Fix]
    F --> G[Test Fix]
    E --> H[Code Review]
    G --> H
    H --> I{Approved?}
    I -->|No| J[Address Feedback]
    J --> H
    I -->|Yes| K[Merge Fix]
    K --> L[Update Tests]
    L --> M[Update Documentation]
```

#### For Minor Bugs

1. If the bug is minor and directly related to the feature being developed:
   - Fix the bug directly in the feature branch
   - Commit the fix with a clear message:
     ```bash
     git commit -m "Fix: Resolve issue with data validation in user registration"
     ```

2. If the bug is in a different feature but within the same work package:
   - Create a bug fix branch from the feature branch:
     ```bash
     git checkout -b bugfix/login-error feature/wp1/user-authentication
     ```
   - Fix the bug and commit the changes
   - Create a pull request to merge the bug fix into the feature branch

#### For Major Bugs or Bugs Affecting Multiple Features

1. Create a dedicated bug fix branch from the main feature branch or work package branch:
   ```bash
   git checkout -b bugfix/critical-data-loss feature/wp1/data-management
   ```

2. Implement the fix and thoroughly test it

3. Create a pull request for code review

4. After approval, merge the bug fix branch into the relevant feature branch(es):
   ```bash
   git checkout feature/wp1/data-management
   git merge --no-ff bugfix/critical-data-loss
   ```

5. If the bug affects multiple work packages, consider merging the fix into `main` and then updating all affected feature branches:
   ```bash
   git checkout main
   git merge --no-ff bugfix/critical-data-loss
   git checkout feature/wp2/data-analysis
   git rebase main
   ```

#### Bug Tracking and Documentation

- Use your issue tracking system (e.g., GitHub Issues, Jira) to document all bugs
- Link commits and pull requests to the relevant bug tickets
- Update test cases to include checks for the resolved bug, preventing regression

#### Code Review for Bug Fixes

- All bug fixes, regardless of size, should go through code review
- For urgent fixes, establish a process for expedited review and merging

#### Regression Testing

- After applying bug fixes, perform regression testing on affected features
- Update automated test suites to include new test cases that check for the resolved bug

#### Communication

- Keep the team informed about significant bugs and their fixes
- In daily stand-ups, briefly mention any bugs that might impact other team members' work

By following these guidelines, you can ensure that bug fixes are properly integrated into your development workflow, maintaining code quality and project stability throughout the development and QA phases.

### 4. Release Preparation and On-site Testing

```mermaid
sequenceDiagram
    participant FBs as Feature Branches
    participant RB as Release Branch
    participant M as Main
    participant P as Production
    
    FBs->>RB: Merge completed features
    Note over RB: Integration testing
    RB->>RB: Bug fixes
    RB->>M: Merge when stable
    M->>M: Tag release
    M->>P: Deploy
    alt Urgent issues
        P->>M: Create hotfix
        M->>P: Deploy hotfix
    end
    M->>FBs: Create new feature branches
```

1. Create a release branch when features are complete:
   ```bash
   git checkout -b release/1.0 main
   ```
2. Merge completed feature branches:
   ```bash
   git checkout release/1.0
   git merge --no-ff feature/wp1/feature-a
   git merge --no-ff feature/wp2/feature-b
   ```
3. Perform final adjustments, integration testing, and bug fixes
4. Update version numbers and metadata

### 5. On-site Testing and Go-Live

1. Deploy the release branch for on-site testing
2. Make necessary adjustments directly on the release branch
3. Merge into `main` after approval:
   ```bash
   git checkout main
   git merge --no-ff release/1.0
   git tag -a v1.0 -m "Version 1.0"
   ```
4. Delete the release branch after merging

### 6. Hotfixes

1. Create a hotfix branch for urgent production fixes:
   ```bash
   git checkout -b hotfix/critical-bug main
   ```
2. Fix the issue and bump the version number
3. Merge the hotfix into `main`:
   ```bash
   git checkout main
   git merge --no-ff hotfix/critical-bug
   git tag -a v1.0.1 -m "Version 1.0.1"
   ```

### 7. System Acceptance and Support

1. Create a support branch after system acceptance:
   ```bash
   git checkout -b support/1.0 v1.0
   ```
2. For non-urgent fixes, create branches from the support branch:
   ```bash
   git checkout -b fix/minor-issue support/1.0
   ```
3. Merge fixes and cherry-pick into `main` if necessary:
   ```bash
   git checkout support/1.0
   git merge --no-ff fix/minor-issue
   git checkout main
   git cherry-pick <commit-hash>
   ```

### 8. New Feature Development for Subsequent Releases

1. Create new feature branches from `main`:
   ```bash
   git checkout -b feature/new-major-feature main
   ```
2. Follow the normal feature development process
3. Create a new release branch when ready for the next release

## Best Practices

- Use pull requests for code review
- Maintain a clean commit history with `git rebase`
- Use meaningful branch names and commit messages
- Tag all releases on the `main` branch
- Document release notes for each version
- Regularly update feature branches with changes from `main`

## Handling Dependencies Between Work Packages

### Dependency Management Process

```mermaid
graph TD
    WP1[Work Package 1] --> F1[Feature 1.1]
    WP1 --> F2[Feature 1.2]
    WP2[Work Package 2] --> F3[Feature 2.1]
    WP2 --> F4[Feature 2.2]
    F1 -.-> F3
    F2 -.-> F4
    WP3[Work Package 3] --> F5[Feature 3.1]
    F3 -.-> F5
    
    classDef independent fill:#1A5EFC,stroke:#006400,stroke-width:2px;
    classDef dependent fill:#9AA0AD,stroke:#8B0000,stroke-width:2px;
    class WP1,F1,F2 independent;
    class WP2,F3,F4,WP3,F5 dependent;
```

1. **Dependency Mapping**
   - Create a dependency map of all work packages and features
   - Identify prerequisites for each component

2. **Branch Creation and Naming**
   - For independent components:
     ```bash
     git checkout -b feature/wp1/independent-feature main
     ```
   - For dependent features:
     ```bash
     git checkout -b feature/wp2/dependent-feature feature/wp1/independent-feature
     ```
   - Use naming convention: `feature/wp2/dependent-feature-on-wp1`

3. **Development Order**
   - Prioritize independent features and prerequisites
   - Begin dependent feature development once prerequisites are stable

4. **Continuous Integration**
   - Regularly update dependent branches:
     ```bash
     git checkout feature/wp2/dependent-feature
     git rebase feature/wp1/independent-feature
     ```
   - Use feature flags or abstract interfaces for parallel development

5. **Code Review and Testing**
   - Review prerequisite features alongside dependent features
   - Test dependent features with their prerequisites

6. **Merging Strategy**
   - Merge prerequisite features first:
     ```bash
     git checkout release/1.0
     git merge --no-ff feature/wp1/independent-feature
     git merge --no-ff feature/wp2/dependent-feature
     ```
   - Resolve conflicts, ensuring proper integration

7. **Release Planning**
   - Group dependent features in release planning
   - Assess impact of delays in prerequisite features

### Best Practices for Managing Dependencies

- Communicate clearly about dependencies among team members
- Use issue tracking systems to link dependent issues/features
- Consider creating integration branches for closely related features
- Regularly assess and update the dependency map
- Use modular design and well-defined interfaces
- Implement automated testing for integration issues
