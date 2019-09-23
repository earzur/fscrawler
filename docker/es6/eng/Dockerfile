FROM toto1310/fscrawler:2.6-es6-nolang

RUN set -ex \
    && apt-get update \
    && apt-get install -y \
        tesseract-ocr \
        tesseract-ocr-eng \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*