# Sync Waves and Hooks Test Cases

This directory contains comprehensive test cases for sync waves and hooks functionality.

## Test Coverage Summary

### Basic Deployment Tests

1. **01-raw** - Raw Kubernetes Manifests
   - Tests basic deployment of raw Kubernetes YAML files
   - Resources: ConfigMap, Secret, ServiceAccount, Deployment, Service, Pod, StatefulSet, DaemonSet, ReplicaSet, CronJob, PersistentVolume, PersistentVolumeClaim
   - No special annotations or hooks
   - Validates basic ServiceDeployment functionality

2. **02-helm** - Helm Chart Deployment
   - Tests deployment using Helm charts
   - Includes Chart.yaml, values.yaml, and templates
   - Validates Helm integration with ServiceDeployment

3. **03-crd** - Custom Resource Definitions
   - Tests deployment of CRDs and their instances
   - Validates custom health checks
   - Tests pipeline gate functionality
   - Scenario: crd-mixed

4. **04-error** - Error Handling
   - Tests error handling and failure scenarios
   - Includes intentionally failing deployments
   - Simulates various deployment failure types
   - Validates error reporting and recovery

5. **06-resync** - Resync Testing
   - Tests periodic resync functionality
   - Validates resync triggers and intervals
   - Uses liquid templates for dynamic configuration
   - Resync interval: 30s

6. **07-mirror** - Mirror/Replication Testing
   - Tests resource mirroring capabilities
   - Validates replication across clusters or namespaces

### Hooks and Sync Waves Tests

7. **08-hooks-basic** - Basic Hook Functionality
   - Tests fundamental hook phases: pre-sync, sync, post-sync, skip
   - Wave values: 0, 1, 2
   - Validates basic hook ordering
   - Tests skip behavior (resources marked as skip are not deployed)
   - Resources: ConfigMap, Secret, Deployment, Job

8. **09-hooks-failure** - Hook Failure Handling
   - Tests sync-fail hook execution when sync resources fail
   - Includes intentionally failing deployment
   - Validates sync-fail hook triggers correctly
   - Tests failure recovery mechanisms

9. **10-hooks-waves** - Complex Wave Ordering
   - Tests complex wave ordering across hook phases
   - Includes negative, zero, and positive wave values
   - Wave range: -1 to 2
   - Validates correct execution order across multiple waves

### Core Functionality Tests

10. **11-helm-hooks** - Helm Hook Support
    - Tests `helm.sh/hook` annotation with: pre-install, pre-upgrade, post-install, post-upgrade
    - Tests `helm.sh/hook-weight` for ordering
    - Resources: ConfigMap, Secret, Job, Deployment

11. **12-argocd-waves** - ArgoCD Sync Wave Support
    - Tests `argocd.argoproj.io/sync-wave` annotation
    - Wave values: -5, 0, 5, 10
    - Resources across all hook phases

12. **13-hook-delete-policies** - Hook Deletion Policies
    - Tests `deployment.plural.sh/sync-hook-delete-policy` with: hook-succeeded, hook-failed
    - Tests `helm.sh/hook-delete-policy`
    - Multiple delete policies per resource
    - Resources with and without delete policies

13. **14-multiple-hooks** - Multiple Hooks Per Resource
    - Tests multiple hook phases on single resources (e.g., pre-sync,post-sync)
    - Combines with delete policies
    - Both plural and helm hook annotations

14. **15-annotation-precedence** - Annotation Priority Order
    - Validates precedence: deployment.plural.sh > argocd.argoproj.io > helm.sh
    - Tests all combinations of annotations
    - Tests fallback to default ordering

15. **16-helm-hook-weight** - Helm Hook Weight Testing
    - Tests negative, zero, and positive weight values
    - Range: -10 to 100
    - Validates ordering within same hook phase

### Advanced Scenario Tests

16. **17-mixed-annotations** - Mixed Annotation Types
    - Combines deployment.plural.sh, helm.sh, and argocd annotations
    - Tests interaction between different annotation types
    - Mixed delete policies

17. **18-complex-scenario** - Real-World Complex Scenario
    - All hook phases: pre-sync, sync, post-sync, sync-fail, skip
    - Wave range: -10 to 20
    - Multiple resource types
    - Delete policies included
    - Simulates production deployment patterns

18. **19-negative-waves** - Negative Wave Values
    - Tests only negative wave values
    - Wave range: -20 to -1
    - Across all hook phases

19. **20-same-wave-ordering** - Same Wave Number Handling
    - Multiple resources with identical wave values
    - Tests deterministic ordering behavior

20. **21-resource-types** - Different Kubernetes Resource Types
    - Tests with: ConfigMap, Secret, ServiceAccount, Deployment, Service, Job, StatefulSet
    - Validates hooks work across all resource types
    - Proper dependency ordering

21. **22-edge-cases** - Edge Case Scenarios
    - Empty wave values
    - Extreme wave numbers (-9999, 9999)
    - All hooks combined on single resource
    - Resources with no annotations

### Additional Combination Tests

22. **23-helm-argocd-combination** - Helm + ArgoCD Without Plural
    - Tests helm.sh/hook with argocd.argoproj.io/sync-wave
    - No deployment.plural.sh annotations
    - Validates precedence works correctly

23. **24-zero-wave-phases** - All Phases at Wave Zero
    - All phases (pre-sync, sync, post-sync) with wave 0
    - Validates phase order is respected when waves are equal

## Annotation Reference

### Hook Phases
- **deployment.plural.sh/sync-hook**: pre-sync, sync, post-sync, sync-fail, skip
- **helm.sh/hook**: pre-install, pre-upgrade, post-install, post-upgrade

### Wave/Weight Annotations
- **deployment.plural.sh/sync-wave**: Integer (highest priority)
- **argocd.argoproj.io/sync-wave**: Integer (medium priority)
- **helm.sh/hook-weight**: Integer (lowest priority)

### Delete Policies
- **deployment.plural.sh/sync-hook-delete-policy**: hook-succeeded, hook-failed
- **helm.sh/hook-delete-policy**: hook-succeeded, hook-failed

## Running Tests

Each test has a corresponding ServiceDeployment in `services/agent/` directory:
- `01-raw.yaml.liquid`
- `02-helm.yaml.liquid`
- `03-crd.yaml.liquid`
- `04-errors.yaml.liquid`
- `06-resync.yaml.liquid`
- `07-mirror.yaml.liquid`
- `08-hooks-basic.yaml.liquid`
- `09-hooks-failure.yaml.liquid`
- `10-hooks-waves.yaml.liquid`
- `11-helm-hooks.yaml.liquid`
- `12-argocd-waves.yaml.liquid`
- ... (etc)

Deploy them to test the functionality in your cluster.

## Test Validation Points

1. **Ordering**: Resources execute in correct wave order within each phase
2. **Phase Sequence**: pre-sync → sync → post-sync (or sync-fail on failure)
3. **Skip Behavior**: Resources marked with 'skip' are not deployed
4. **Delete Policies**: Resources deleted according to policy after hook completion
5. **Multiple Hooks**: Resources execute in all specified phases
6. **Precedence**: Correct annotation precedence order is followed
7. **No Annotation**: Resources without annotations use default ordering
8. **Error Handling**: Failures are properly reported and sync-fail hooks execute
9. **Resync**: Resources resync at configured intervals
10. **Helm Integration**: Helm charts deploy correctly with hook support
