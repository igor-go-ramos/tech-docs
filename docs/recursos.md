## :material-hammer-wrench: Ferramentas
* Syncthing
* WireGuard
* CasaOS
* WifiMan
* iPerf
* Apps Script
* Autounattend (windows 10 e 11)
* testdisk
* Upscayl
* Ventoy
* VDO.Ninja
* OBS Studio
---
## :fontawesome-solid-link: Links Úteis
* [Configuração de SMTP](https://docs.google.com/document/d/16m1ZEhAKygJpjgrJknBPUb9b3rjLxdhhCHZHBljYynk/edit?usp=sharing)
* [Unattended install generator (Windows)](https://schneegans.de/windows/unattend-generator/)
* [Instalação do Debian (com RAID)](https://youtu.be/ykzR3i-pToY?si=CDfnqpJHMjH8710p)
* [Conexão de rede privada (Cloudflare)](https://developers.cloudflare.com/learning-paths/replace-vpn/connect-private-network/cloudflared/)
* [MkDocs Icons](https://squidfunk.github.io/mkdocs-material/reference/icons-emojis/)
* [Samba AD DC no Ubuntu](https://youtu.be/KT6O-TfJ41g?si=R7DPS4_01D_pCXVs)
---
## :fontawesome-solid-code: Scripts

???tip "Extração de Áudio"
    ```bash
    #!/usr/bin/env bash

    set -euo pipefail

    INPUT_DIR="."
    OUTPUT_DIR="."
    RECURSIVE=false
    FORCE=false

    usage() {
        cat <<EOF
    Usage: $(basename "$0") [OPTIONS]

    Extract the original audio stream from MP4 files without re-encoding.

    Options:
      -i, --input-dir DIR     Directory containing MP4 files (default: .)
      -o, --output-dir DIR    Directory where extracted audio is saved (default: .)
      -r, --recursive         Search input directory recursively
      -f, --force             Overwrite existing output files
      -h, --help              Show this help message
    EOF
    }

    while [[ $# -gt 0 ]]; do
        case "$1" in
            -i|--input-dir)
                [[ $# -ge 2 ]] || { echo "Missing argument for $1"; exit 1; }
                INPUT_DIR="$2"
                shift 2
                ;;
            -o|--output-dir)
                [[ $# -ge 2 ]] || { echo "Missing argument for $1"; exit 1; }
                OUTPUT_DIR="$2"
                shift 2
                ;;
            -r|--recursive)
                RECURSIVE=true
                shift
                ;;
            -f|--force)
                FORCE=true
                shift
                ;;
            -h|--help)
                usage
                exit 0
                ;;
            *)
                echo "Unknown option: $1"
                usage
                exit 1
                ;;
        esac
    done

    [[ -d "$INPUT_DIR" ]] || {
        echo "Error: Input directory '$INPUT_DIR' does not exist."
        exit 1
    }

    mkdir -p "$OUTPUT_DIR"

    extract_audio() {
        local input="$1"

        local rel="${input#$INPUT_DIR/}"
        local dir
        dir=$(dirname "$rel")

        local filename
        filename=$(basename "$input")
        local stem="${filename%.*}"

        local codec
        codec=$(ffprobe -v error \
            -select_streams a:0 \
            -show_entries stream=codec_name \
            -of default=noprint_wrappers=1:nokey=1 \
            "$input")

        local ext

        case "$codec" in
            aac)     ext="m4a" ;;
            mp3)     ext="mp3" ;;
            flac)    ext="flac" ;;
            opus)    ext="opus" ;;
            vorbis)  ext="ogg" ;;
            alac)    ext="m4a" ;;
            pcm_*)   ext="wav" ;;
            *)
                echo "Skipping '$rel': unsupported codec '$codec'"
                return
                ;;
        esac

        local outdir="$OUTPUT_DIR"
        [[ "$dir" != "." ]] && outdir="$OUTPUT_DIR/$dir"

        mkdir -p "$outdir"

        local output="$outdir/$stem.$ext"

        if [[ -f "$output" && "$FORCE" == false ]]; then
            echo "Skipping '$rel': '$output' already exists."
            return
        fi

        echo "Extracting:"
        echo "  Input : $rel"
        echo "  Output: $output"



        ffmpeg_args=(
            -hide_banner
            -loglevel error
        )

        if $FORCE; then
          ffmpeg_args+=(-y)
        else
            ffmpeg_args+=(-n)
        fi

        ffmpeg "${ffmpeg_args[@]}" \
            -i "$input" \
            -vn \
            -map 0:a:0 \
            -c:a copy \
            -map_metadata 0 \
            "$output" 
    }

    if $RECURSIVE; then
        while IFS= read -r -d '' file; do
            extract_audio "$file"
        done < <(find "$INPUT_DIR" -type f -iname "*.mp4" -print0)
    else
        shopt -s nullglob
        for file in "$INPUT_DIR"/*.mp4; do
            extract_audio "$file"
        done
    fi

    echo "Done!"
    ```