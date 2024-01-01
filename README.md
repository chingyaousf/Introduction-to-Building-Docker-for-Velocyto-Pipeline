## **Introduction-to-Building-Docker-for-Velocyto-Pipeline**

This repository provides a Docker image for running the Velocyto pipeline on Single Cell Gene Expression Datasets generated by Cell Ranger from 10X Genomics. Velocyto is a powerful tool designed for the analysis of single-cell RNA-seq data, particularly for understanding RNA dynamics at the single-cell level.

### **Velocyto Overview**

**Velocyto** is an open-source Python library developed by the Velocyto Team. It specializes in the analysis of single-cell RNA sequencing (scRNA-seq) data, providing insights into RNA splicing dynamics, RNA velocity, and cell state transitions. Here are some key features and concepts related to Velocyto:

-   **RNA Velocity:** Velocyto is known for its ability to estimate the future state of individual cells in terms of gene expression. This concept, known as RNA velocity, is a powerful tool for understanding the dynamics of gene expression changes and predicting the future state of individual cells.

-   **Single-Cell RNA-seq:** Velocyto is specifically tailored for the analysis of single-cell RNA-seq data, which captures the transcriptomic profile of individual cells. This technology has revolutionized our ability to study gene expression at the single-cell level, providing a more detailed and nuanced understanding of cellular heterogeneity.

-   **Cell Ranger Compatibility:** Velocyto is often used in conjunction with data processed by Cell Ranger, a popular software suite provided by 10X Genomics for analyzing single-cell gene expression data.

### **About the Dataset**

The Docker image provided in this repository is tailored for the analysis of a specific dataset: Frozen Peripheral Blood Mononuclear Cells (PBMCs) from Donor A. The dataset was generated using Cell Ranger 1.1.0, sequenced on an Illumina NextSeq 500 High Output platform, and published on July 24, 2016.

### **Dockerfile**

``` dockerfile
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
COPY app_03/sample/barcodes.tsv /app/sample/
COPY app_03/sample/frozen_pbmc_donor_a_possorted_genome_bam.bam /app/sample/
COPY app_03/genes.gtf /app/
#COPY app/sample/cellsorted_frozen_pbmc_donor_a_possorted_genome_bam.bam /app/sample/

# Sort the BAM file using samtools
#RUN samtools sort -o /app/sample/cellsorted_frozen_pbmc_donor_a_possorted_genome_bam.bam /app/sample/frozen_pbmc_donor_a_possorted_genome_bam.bam
RUN samtools sort -l 7 -m 1024M -t CB -O BAM -@ 4 -o /app/sample/cellsorted_frozen_pbmc_donor_a_possorted_genome_bam.bam /app/sample/frozen_pbmc_donor_a_possorted_genome_bam.bam

ENTRYPOINT ["velocyto"]
#CMD ["--help"]

# Specify the default command
CMD ["run", "-b", "sample/barcodes.tsv", "-o", "velocyto_output", "sample/frozen_pbmc_donor_a_possorted_genome_bam.bam", "genes.gtf"]
```

### **Running the Docker Container Using Docker Desktop and Windows cmd**:

To embark on Building Docker for Velocyto, we encourage you to install Docker Desktop on your system. Once you've installed Docker Desktop, you can build the Docker image and run the container with the following simple steps:

#### **Velocyto Docker Setup:**

**1. Install Docker Desktop**: Download and install Docker Desktop for your operating system from the official Docker website ([https://www.docker.com/products/docker-desktop).](https://www.docker.com/products/docker-desktop).)

**2. Build the Docker Image**:

-   Clone the Docker project repository from its source.

    Click github.dev

    ![](https://github.com/chingyaousf/Introduction-to-Building-Docker-for-Velocyto-Pipeline/blob/main/data/Velocyto%20Docker%20Desktop%20images_01.png?raw=true)

    Download Docker folder

    ![](https://github.com/chingyaousf/Introduction-to-Building-Docker-for-Velocyto-Pipeline/blob/main/data/Velocyto%20Docker%20Desktop%20images_02.png?raw=true)

-   Open Windows command prompt and navigate to the directory containing the Dockerfie.

```         
C:\>cd C:\Users\Administrator\Desktop\Docker
```

-   Use the `docker build` command to build the Docker image. For example:

```         
docker build -t velocyto_app .
```

**3. Run the Docker Container**:

-   Once the Docker image is successfully built, you can run the image using **Docker Desktop**.

    ![](https://github.com/chingyaousf/Introduction-to-Building-Docker-for-Velocyto-Pipeline/blob/main/data/Velocyto%20Docker%20Desktop%20images_03.png?raw=true)

    ![](https://github.com/chingyaousf/Introduction-to-Building-Docker-for-Velocyto-Pipeline/blob/main/data/Velocyto%20Docker%20Desktop%20images_04.png?raw=true)

    ![](https://github.com/chingyaousf/Introduction-to-Building-Docker-for-Velocyto-Pipeline/blob/main/data/Velocyto%20Docker%20Desktop%20images_05.png?raw=true)

-   Run the image using **Windows cmd**

    command:

    ```         
    docker run -it velocyto_app:latest
    ```

Once the container has finished running, you should be able to access **loom** output file inside the **'app/velocyto_output'** folder of the downloaded Docker repository.

**4. Use Docker Compose for Building and Running**:

-   Create a Docker Compose YAML file in the Docker project directory (already included in the Docker file).

-   Inside the YAML file, specify the services, image names, ports, and other configurations.

-   Use **`docker-compose build`** to build the image based on the YAML file. You can run this command on Windows command prompt whenever the Dockerfile is modified to update the image.

-   Use **`docker-compose up`** to run the container based on the built image. This is also used to start the container whenever needed.

-   To remove the container and associated resources, use **`docker-compose down`**.

The Docker container is configured to run Velocyto app, ensuring that you can efficiently produce **loom** output file with ease and consistency. This setup allows for conveniently producing loom output file. Remember that you can run `docker-compose build` or `docker-compose up` every time you modify the Dockerfile to update the image and keep your analysis environment up to date. Please refer to the **Docker Desktop Learning Center** and **Docker Compose documentation** for comprehensive guidance on using these tools effectively.

### **Running the Docker Container in HPC**:

-   Clone the Docker project repository from its source.

-   Refer instruction Podman (Docker), and Singularity (Apptainer) HPC Guide

    <https://github.com/chingyaousf/Simple-Podman-Docker-and-Singularity-Apptainer-HPC-Guide>

**Note:** Ensure that the necessary data files are available in the appropriate paths inside the container as specified in the Dockerfile.

Feel free to explore and adapt the Dockerfile and associated commands based on your specific requirements and dataset.

### Blog

### Data source

<https://support.10xgenomics.com/single-cell-gene-expression/datasets/1.1.0/frozen_pbmc_donor_a>

### References

<https://velocyto.org/velocyto.py/index.html#>

<https://github.com/chingyaousf/Simple-Podman-Docker-and-Singularity-Apptainer-HPC-Guide>

<https://github.com/hisplan/docker-velocyto/blob/master/Dockerfile>
