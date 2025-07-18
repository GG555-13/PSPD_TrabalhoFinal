FROM ubuntu:22.04

# Instalar dependências com otimizações
RUN apt-get update && apt-get install -y \
    gcc \
    libc6-dev \
    libgomp1 \
    libcurl4-openssl-dev \
    libjson-c-dev \
    openmpi-bin \
    openmpi-common \
    libopenmpi-dev \
    numactl \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

COPY src/socket/socket_server.c /app/socket_server.c
COPY src/core/jogodavida_openmp_mpi.c /app/jogodavida_openmp_mpi.c

ARG OPTIMIZATION_FLAGS="-O3 -march=native -mtune=native -flto -funroll-loops"

RUN echo "Compilando com flags: $OPTIMIZATION_FLAGS" && \
    gcc $OPTIMIZATION_FLAGS -o socket_server socket_server.c \
        -lcurl -ljson-c -lpthread && \
    mpicc $OPTIMIZATION_FLAGS -o jogodavida_openmp_mpi jogodavida_openmp_mpi.c \
        -fopenmp -lm

RUN mkdir -p /tmp /dev/shm && \
    chmod 1777 /tmp /dev/shm

ENV OMP_NUM_THREADS=4 \
    OMP_PROC_BIND=true \
    OMP_PLACES=cores \
    MALLOC_ARENA_MAX=2 \
    MALLOC_MMAP_THRESHOLD_=131072

RUN echo '#!/bin/bash\n\
exec mpirun -np 2 --allow-run-as-root ./jogodavida_openmp_mpi "$@"\n\
' > /app/entrypoint.sh && chmod +x /app/entrypoint.sh

EXPOSE 8080

ENTRYPOINT ["/app/entrypoint.sh"]

CMD []