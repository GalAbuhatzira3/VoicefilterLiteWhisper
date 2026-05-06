================================================================
STEP 1 — cd into the project root
==================================================================
The "project root" is the directory that contains environment.yml,
scripts/, whisper_vf/, runs/, exp/, cache/.

    cd path/to/experimental_whisper_windows_ready

Every command below is relative to that directory. Do not cd anywhere else.

==================================================================
STEP 2 — install the environment
==================================================================
Use ONE of:

    conda env create -f environment.yml        # GPU
    conda env create -f environment_cpu.yml    # CPU
    pip install -r requirements.txt

==================================================================
STEP 3 — download LibriSpeech (raw audio)
==================================================================
Pulls the three subsets used by Libri2Mix from openslr.org, extracts them
into data_librimix/LibriSpeech/. Roughly ~9 GB on disk after extraction.
Resumes partial downloads automatically.

    python scripts/download_librispeech_for_librimix.py

By default this downloads:
    train-clean-100, dev-clean, test-clean

Result:
    data_librimix/LibriSpeech/train-clean-100/
    data_librimix/LibriSpeech/dev-clean/
    data_librimix/LibriSpeech/test-clean/

==================================================================
STEP 4 — generate Libri2Mix (mixed audio)
==================================================================
Mixes pairs of LibriSpeech utterances using the official Libri2Mix
metadata CSVs (downloaded automatically from JorisCos/LibriMix on GitHub),
producing the s1/s2/mix_clean wav files at 16 kHz, mode=max.

    python scripts/generate_libri2mix_clean_only.py

Defaults (matching this run):
    --librispeech_root data_librimix/LibriSpeech
    --out_root         data_librimix
    --metadata_dir     data_librimix/metadata/Libri2Mix
    --splits           train-100 dev test
    --sample_rate      16000
    --mode             max
    --num_workers      4

Result:
    data_librimix/Libri2Mix/wav16k/max/train-100/{s1,s2,mix_clean}/
    data_librimix/Libri2Mix/wav16k/max/dev/{s1,s2,mix_clean}/
    data_librimix/Libri2Mix/wav16k/max/test/{s1,s2,mix_clean}/

Counts you should see when training launches:
    train-100: 27800 cases   dev: 6000 cases   test: 6000 cases

==================================================================
STEP 5 — precompute speaker embeddings (cache)
==================================================================
Each block below sets PYTHONPATH then runs the precompute script. Pick
the block matching your shell.

----- Linux / macOS / Git-Bash / WSL -----
export PYTHONPATH="$PWD"

python scripts/precompute_librimix_speaker_embeddings.py \
    --data_root data_librimix/Libri2Mix/wav16k/max \
    --splits train-100 dev test \
    --mix_types mix_clean \
    --num_speakers 2 \
    --speaker_backend speechbrain \
    --speaker_device cpu \
    --speaker_cache cache/whisper_vf_speaker_librimix

----- Windows PowerShell -----
$env:PYTHONPATH = (Get-Location).Path

python scripts/precompute_librimix_speaker_embeddings.py `
    --data_root data_librimix/Libri2Mix/wav16k/max `
    --splits train-100 dev test `
    --mix_types mix_clean `
    --num_speakers 2 `
    --speaker_backend speechbrain `
    --speaker_device cpu `
    --speaker_cache cache/whisper_vf_speaker_librimix

----- Windows cmd.exe -----
set PYTHONPATH=%cd%

python scripts/precompute_librimix_speaker_embeddings.py ^
    --data_root data_librimix/Libri2Mix/wav16k/max ^
    --splits train-100 dev test ^
    --mix_types mix_clean ^
    --num_speakers 2 ^
    --speaker_backend speechbrain ^
    --speaker_device cpu ^
    --speaker_cache cache/whisper_vf_speaker_librimix

Result:
    cache/whisper_vf_speaker_librimix/*.npy   (one file per unique speaker
                                              enrollment, hundreds of files)

speaker_device=cpu is intentional — speechbrain ECAPA on cpu was more
reliable than gpu and very fast anyway.
==================================================================
STEP 6 — train
==================================================================
The arguments are identical on every OS; the only thing that differs is
how the shell continues a long line.

----- Linux / macOS / Git-Bash / WSL -----
mkdir -p runs/2026-04-28_libri2mix_train100_seg1200_less_handcuffed_b1_acc8

python -u scripts/train_whisper_vf.py \
    --data_root data_librimix/Libri2Mix/wav16k/max \
    --train_split train-100 \
    --val_split dev \
    --mix_type mix_clean \
    --epochs 20 \
    --batch_size 1 \
    --segment_frames 1200 \
    --lr 0.0007 \
    --min_lr 0.00001 \
    --weight_decay 0.00001 \
    --grad_clip 5.0 \
    --accum_steps 8 \
    --hidden_size 384 \
    --num_layers 3 \
    --dropout 0.05 \
    --mode residual \
    --max_residual 0.35 \
    --speaker_backend speechbrain \
    --speaker_device cpu \
    --speaker_cache cache/whisper_vf_speaker_librimix \
    --alpha 4.0 \
    --identity_weight 0.01 \
    --smoothness_weight 0.005 \
    --aux_weight 0.05 \
    --device cuda \
    --feature_device auto \
    --seed 1337 \
    --early_stop_patience 6 \
    --early_stop_min_delta 0 \
    --num_workers 0 \
    --prefetch_factor 4 \
    --debug_train_metrics_every 500 \
    --exp_dir exp/whisper_vf_libri2mix_train100_seg1200_less_handcuffed_b1_acc8 \
    2>&1 | tee runs/2026-04-28_libri2mix_train100_seg1200_less_handcuffed_b1_acc8/train.log

----- Windows PowerShell -----
New-Item -ItemType Directory -Path runs/2026-04-28_libri2mix_train100_seg1200_less_handcuffed_b1_acc8 -Force | Out-Null

python -u scripts/train_whisper_vf.py `
    --data_root data_librimix/Libri2Mix/wav16k/max `
    --train_split train-100 `
    --val_split dev `
    --mix_type mix_clean `
    --epochs 20 `
    --batch_size 1 `
    --segment_frames 1200 `
    --lr 0.0007 `
    --min_lr 0.00001 `
    --weight_decay 0.00001 `
    --grad_clip 5.0 `
    --accum_steps 8 `
    --hidden_size 384 `
    --num_layers 3 `
    --dropout 0.05 `
    --mode residual `
    --max_residual 0.35 `
    --speaker_backend speechbrain `
    --speaker_device cpu `
    --speaker_cache cache/whisper_vf_speaker_librimix `
    --alpha 4.0 `
    --identity_weight 0.01 `
    --smoothness_weight 0.005 `
    --aux_weight 0.05 `
    --device cuda `
    --feature_device auto `
    --seed 1337 `
    --early_stop_patience 6 `
    --early_stop_min_delta 0 `
    --num_workers 0 `
    --prefetch_factor 4 `
    --debug_train_metrics_every 500 `
    --exp_dir exp/whisper_vf_libri2mix_train100_seg1200_less_handcuffed_b1_acc8 `
    2>&1 | Tee-Object -FilePath runs/2026-04-28_libri2mix_train100_seg1200_less_handcuffed_b1_acc8/train.log

----- Windows cmd.exe -----
if not exist runs\2026-04-28_libri2mix_train100_seg1200_less_handcuffed_b1_acc8 mkdir runs\2026-04-28_libri2mix_train100_seg1200_less_handcuffed_b1_acc8

python -u scripts/train_whisper_vf.py ^
    --data_root data_librimix/Libri2Mix/wav16k/max ^
    --train_split train-100 --val_split dev --mix_type mix_clean ^
    --epochs 20 --batch_size 1 --segment_frames 1200 ^
    --lr 0.0007 --min_lr 0.00001 --weight_decay 0.00001 ^
    --grad_clip 5.0 --accum_steps 8 ^
    --hidden_size 384 --num_layers 3 --dropout 0.05 ^
    --mode residual --max_residual 0.35 ^
    --speaker_backend speechbrain --speaker_device cpu ^
    --speaker_cache cache/whisper_vf_speaker_librimix ^
    --alpha 4.0 --identity_weight 0.01 --smoothness_weight 0.005 --aux_weight 0.05 ^
    --device cuda --feature_device auto ^
    --seed 1337 ^
    --early_stop_patience 6 --early_stop_min_delta 0 ^
    --num_workers 0 --prefetch_factor 4 ^
    --debug_train_metrics_every 500 ^
    --exp_dir exp/whisper_vf_libri2mix_train100_seg1200_less_handcuffed_b1_acc8 ^
    > runs\2026-04-28_libri2mix_train100_seg1200_less_handcuffed_b1_acc8\train.log 2>&1

==================================================================
STEP 7 — evaluate a specific epoch (fixed-seed dev500)
==================================================================
EPOCH (zero-padded, e.g. 05) is the number of the epoch to evaluate.

----- Linux / macOS / Git-Bash / WSL -----
EPOCH=05
SNAP=runs/2026-04-28_libri2mix_train100_seg1200_less_handcuffed_b1_acc8/snapshots
EVAL=runs/2026-04-28_libri2mix_train100_seg1200_less_handcuffed_b1_acc8/eval_epoch${EPOCH}_dev500_b8_seed42
mkdir -p "$SNAP" "$EVAL"
cp exp/whisper_vf_libri2mix_train100_seg1200_less_handcuffed_b1_acc8/epoch${EPOCH}.pt \
   "$SNAP/epoch${EPOCH}_seed42.pt"

python -u scripts/eval_whisper_vf.py \
    --data_root data_librimix/Libri2Mix/wav16k/max \
    --librispeech_root data_librimix/LibriSpeech \
    --split dev \
    --mix_type mix_clean \
    --vf_checkpoint "$SNAP/epoch${EPOCH}_seed42.pt" \
    --whisper_model small.en \
    --batch_size 8 \
    --max_examples 500 \
    --seed 42 \
    --device cuda \
    --speaker_device cpu \
    --speaker_cache cache/whisper_vf_speaker_librimix \
    --eval_wrong_dvec \
    --output_dir "$EVAL" \
    2>&1 | tee "$EVAL/eval.log"

----- Windows PowerShell -----
$EPOCH = "05"
$SNAP  = "runs/2026-04-28_libri2mix_train100_seg1200_less_handcuffed_b1_acc8/snapshots"
$EVAL  = "runs/2026-04-28_libri2mix_train100_seg1200_less_handcuffed_b1_acc8/eval_epoch${EPOCH}_dev500_b8_seed42"
New-Item -ItemType Directory -Path $SNAP, $EVAL -Force | Out-Null
Copy-Item "exp/whisper_vf_libri2mix_train100_seg1200_less_handcuffed_b1_acc8/epoch${EPOCH}.pt" `
          "$SNAP/epoch${EPOCH}_seed42.pt" -Force

python -u scripts/eval_whisper_vf.py `
    --data_root data_librimix/Libri2Mix/wav16k/max `
    --librispeech_root data_librimix/LibriSpeech `
    --split dev `
    --mix_type mix_clean `
    --vf_checkpoint "$SNAP/epoch${EPOCH}_seed42.pt" `
    --whisper_model small.en `
    --batch_size 8 `
    --max_examples 500 `
    --seed 42 `
    --device cuda `
    --speaker_device cpu `
    --speaker_cache cache/whisper_vf_speaker_librimix `
    --eval_wrong_dvec `
    --output_dir $EVAL `
    2>&1 | Tee-Object -FilePath "$EVAL/eval.log"

----- Windows cmd.exe -----
set EPOCH=05
set SNAP=runs\2026-04-28_libri2mix_train100_seg1200_less_handcuffed_b1_acc8\snapshots
set EVAL=runs\2026-04-28_libri2mix_train100_seg1200_less_handcuffed_b1_acc8\eval_epoch%EPOCH%_dev500_b8_seed42
if not exist %SNAP% mkdir %SNAP%
if not exist %EVAL% mkdir %EVAL%
copy /Y exp\whisper_vf_libri2mix_train100_seg1200_less_handcuffed_b1_acc8\epoch%EPOCH%.pt ^
        %SNAP%\epoch%EPOCH%_seed42.pt

python -u scripts/eval_whisper_vf.py ^
    --data_root data_librimix/Libri2Mix/wav16k/max ^
    --librispeech_root data_librimix/LibriSpeech ^
    --split dev --mix_type mix_clean ^
    --vf_checkpoint %SNAP%\epoch%EPOCH%_seed42.pt ^
    --whisper_model small.en ^
    --batch_size 8 --max_examples 500 ^
    --seed 42 ^
    --device cuda --speaker_device cpu ^
    --speaker_cache cache/whisper_vf_speaker_librimix ^
    --eval_wrong_dvec ^
    --output_dir %EVAL% ^
    > %EVAL%\eval.log 2>&1
