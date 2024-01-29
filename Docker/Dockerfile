FROM ubuntu:22.04

# Maintainer information
LABEL maintainer="Tim Shaw <timothy.shaw@moffitt.org>" \
      version.velocyto="0.17.17" \
      version.samtools="1.18" \
      source.velocyto="https://github.com/velocyto-team/velocyto.py/releases/tag/0.17.17" \
      source.samtools="https://github.com/samtools/samtools/releases/tag/1.18"

# Set the working directory inside the container
WORKDIR /app

ENV VELOCYTO_VERSION 0.17.17
ENV SAMTOOLS_VERSION 1.18

# required by click
ENV LC_ALL C.UTF-8
ENV LANG C.UTF-8

# update package manager, build essentials, and install python 3
RUN apt-get update \
    && apt-get install --yes build-essential python3 python3-pip

# install dependency required by samtools
RUN apt-get install --yes wget libncurses5-dev zlib1g-dev libbz2-dev liblzma-dev

# install samtools (which is required by velocyto)
RUN cd /tmp \
    && wget https://github.com/samtools/samtools/releases/download/${SAMTOOLS_VERSION}/samtools-${SAMTOOLS_VERSION}.tar.bz2 \
    && tar xvjf samtools-${SAMTOOLS_VERSION}.tar.bz2 \
    && cd samtools-${SAMTOOLS_VERSION} \
    && ./configure --prefix=/usr/local \
    && make \
    && make install \
    && cd / && rm -rf /tmp/samtools-${SAMTOOLS_VERSION}

# install python packages required by velocyto
RUN pip3 install numpy scipy cython numba matplotlib scikit-learn h5py click

# install velocyto
RUN pip3 install velocyto==${VELOCYTO_VERSION}

# Copy script and data files into the container
COPY app/sample/barcodes.tsv /app/sample/
COPY app/sample/frozen_pbmc_donor_a_possorted_genome_bam.bam /app/sample/
COPY app/genes.gtf /app/
#COPY app/sample/cellsorted_frozen_pbmc_donor_a_possorted_genome_bam.bam /app/sample/

# Sort the BAM file using samtools
#RUN samtools sort -o /app/sample/cellsorted_frozen_pbmc_donor_a_possorted_genome_bam.bam /app/sample/frozen_pbmc_donor_a_possorted_genome_bam.bam
RUN samtools sort -l 7 -m 1024M -t CB -O BAM -@ 4 -o /app/sample/cellsorted_frozen_pbmc_donor_a_possorted_genome_bam.bam /app/sample/frozen_pbmc_donor_a_possorted_genome_bam.bam

ENTRYPOINT ["velocyto"]
#CMD ["--help"]

# Specify the default command
CMD ["run", "-b", "sample/barcodes.tsv", "-o", "velocyto_output", "sample/frozen_pbmc_donor_a_possorted_genome_bam.bam", "genes.gtf"]

