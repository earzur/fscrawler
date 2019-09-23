FROM toto1310/fscrawler:2.7-SNAPSHOT-es7-nolang

RUN set -ex \
    && apt-get update \
    && apt-get install -y \
        tesseract-ocr \
        tesseract-ocr-jpn \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*
