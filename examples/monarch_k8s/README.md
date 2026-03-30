### Monarch-TorchFT-TorchTitan Distributed Training Orchestrator

#### Overview
This script orchestrates fault-tolerant distributed training using TorchTitan and TorchMonarch
frameworks. It manages multiple training replicas across Kubernetes-scheduled pods
with automatic failure recovery and TorchFT lighthouse coordination.

##### PREREQUISITES
- Running inside a Kubernetes cluster with the MonarchMesh operator installed
- The `kubernetes` Python package installed (`pip install kubernetes`)
- A ServiceAccount with RBAC permissions to manage MonarchMesh custom resources
- TorchTitan training configuration file in script directory (debug_model.toml)
- A training dataset (c4_test) and tokenizer in script directory

##### MODES OF OPERATION
This example supports two modes:

**Attach-only mode** (default): Connects to pre-provisioned pods discovered via label selectors.
Pods must already exist in the specified namespace.

**Provisioning mode**: Creates MonarchMesh CRDs that the operator provisions automatically.
Enabled by passing `--image` (and optionally `--gpu-resources`).

##### USAGE
    python train_distributed.py --help

    Attach-only mode with 2 replicas (pods must already exist):
        python train_distributed.py --namespace monarch-tests

    Provisioning mode with 2 replicas, each with 8 GPUs:
        python train_distributed.py --namespace monarch-tests \
            --image ghcr.io/meta-pytorch/monarch:latest \
            --gpu-resources 8

    Custom configuration:
        python train_distributed.py --namespace monarch-tests \
            --replica-count 3 --gpu-per-node 8 \
            --host-per-replica 2 --training-steps 100 \
            --image my-image:tag --gpu-resources 8

    With remote TorchFT lighthouse:
        python train_distributed.py --namespace monarch-tests --remote-lighthouse

    With pod readiness timeout:
        python train_distributed.py --namespace monarch-tests --timeout 300

##### KEY COMPONENTS
- LighthouseActor: Coordination server for fault tolerance
- TrainingActor: Individual trainer processes
- ReplicaActor: Manages groups of trainers
- OrchestrationManager: Top-level orchestration and failure recovery
- FailureController: Optional, periodically injects random failures into trainer processes

##### FAILURE RECOVERY
- Automatic retry with configurable delays (PER_ATTEMPT_DELAY)
- New allocations after repeated failures (PROC_ATTEMPTS)
- Maximum attempts per replica (MAX_ATTEMPT)

##### OUTPUT
- Training outputs saved to ./outputs directory
- Logs streamed from all distributed processes
- TensorBoard metrics enabled by default

##### CLEANUP
All Kubernetes MonarchMesh resources are automatically cleaned up at script completion.
