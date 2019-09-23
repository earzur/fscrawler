FROM toto1310/fscrawler:2.7-SNAPSHOT-es5-nolang

RUN set -ex \
    && apt-get update \
    && apt-get install -y \
        tesseract-ocr \
        tesseract-ocr-fra \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*
