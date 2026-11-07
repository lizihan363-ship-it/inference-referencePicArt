DEADiff data1200, generation seed=0.
1200 final generated images = 60 references x 20 target prompts.
E21 and E30 use updated references and 40 rerun outputs; the other 1160 outputs retain their original SHA256.
images/: generated 512x512 PNG files. references/: 60 current original references.
portable_manifest.json: explicit relative image paths, prompts, reference IDs, seed, and SHA256.
generation_config.json: generation settings and model/source hashes with machine paths removed.
SHA256SUMS and archive_content_manifest.json cover all image and metadata payloads, excluding these two indexes.
No model checkpoints, environments, machine paths, credentials, or logs are included.
