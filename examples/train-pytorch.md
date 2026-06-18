# Example: fine-tune a model on an L4

End-to-end recipe for training a PyTorch model on a cloud GPU and bringing the
checkpoints back.

```bash
# 0. one-time: gcloud auth login && gcloud config set project YOUR_PROJECT_ID

# 1. create a 24GB L4 box
GPU=nvidia-l4 ZONE=us-central1-a NAME=trainer gpu-vm create
NAME=trainer gpu-vm status            # wait until nvidia-smi shows the L4

# 2. push your repo (commit first — git archive only ships committed files)
NAME=trainer gpu-vm push ~/code/my-model

# 3. set up the environment (run as a background step; installs take minutes)
NAME=trainer gpu-vm ssh "cd my-model && python3 -m venv venv && \
  ./venv/bin/pip install -r requirements.txt"

# 4. launch training in tmux so it survives disconnects
NAME=trainer gpu-vm run \
  "cd my-model && ./venv/bin/python train.py --epochs 10 --out runs/exp1" \
  exp1

# 5. monitor
NAME=trainer gpu-vm logs exp1.log     # live logs
NAME=trainer gpu-vm status            # GPU utilization
NAME=trainer gpu-vm wait exp1         # block until done

# 6. retrieve results, then stop billing
NAME=trainer gpu-vm pull my-model/runs/exp1 ./exp1
NAME=trainer gpu-vm stop              # or: delete
```

## Tips

- **Need more VRAM?** `GPU=nvidia-a100-80gb ZONE=us-central1-c gpu-vm create`.
- **Cheap and interruptible?** Add `SPOT=1` to `create` and checkpoint often.
- **Big datasets?** Prefer a GCS bucket (`gsutil cp`) over the boot disk.
- **Driving it from an AI agent?** Every command returns the VM's stdout to your
  terminal, so the agent can read results directly — use `run`/`wait` for long
  jobs and `status`/`logs` to poll.
