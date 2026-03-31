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
    Custom configuration:
        python train_distributed.py \
            --num-replicas 4 \
            --gpus-per-host 4 \
            --training-steps 100 \
            --model-config debug_model.toml \
            --dataset-path c4_test \
            --tokenizer-path debug_tokenizer \
            --with-failures True \
            --namespace monarch-tests-titan-torchft \
            --image ocir.ap-sydney-1.oci.oraclecloud.com/iduyx1qnmway/meta-pytorch/monarch:0.4.0rc1-cuda12.8

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
