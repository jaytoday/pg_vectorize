FROM postgres:16-bookworm as builder

ARG VECTOR_VER=0.7.1
ARG CRON_VER=1.6.2
ARG PGRX_VER=0.11.3

RUN apt-get update \
    && apt-get install -y \
	ca-certificates \
	clang \
	curl \
	gcc \
	git \
	libssl-dev \
	make \
	pkg-config \
	postgresql-server-dev-16

WORKDIR /vectorize

# install pgvector
RUN git clone https://github.com/pgvector/pgvector.git && \
	cd pgvector && \
	git fetch origin v$VECTOR_VER && \
	git checkout v$VECTOR_VER && \
	make && make install && \
    cd .. && rm -rf pgvector

# install pg_cron
RUN git clone https://github.com/citusdata/pg_cron.git --branch v$CRON_VER && \
	cd pg_cron && \
	make && make install && \
    cd .. && rm -rf pg_cron

# install pgmq
RUN git clone https://github.com/tembo-io/pgmq.git && \
	cd pgmq && \
	make && \
	make install && \
	cd .. && rm -rf pgmq

# Install Rust dependencies
RUN curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y
RUN $HOME/.cargo/bin/rustup default stable

# install pgrx
RUN $HOME/.cargo/bin/cargo install cargo-pgrx --version=$PGRX_VER --locked
RUN $HOME/.cargo/bin/cargo pgrx init --pg16 $(which pg_config)

# install pg-vectorize
COPY . .
RUN cd extension && \
	$HOME/.cargo/bin/cargo pgrx install --pg-config=$(which pg_config)

FROM postgres:16-bookworm

COPY --from=builder /usr/share/postgresql/16/extension /usr/share/postgresql/16/extension
COPY --from=builder /usr/lib/postgresql/16/lib /usr/lib/postgresql/16/lib

RUN apt-get update \
    && apt-get install -y ca-certificates

COPY images/vectorize-pg/postgresql.conf /usr/share/postgresql/16/postgresql.conf.sample

USER postgres
CMD ["postgres"]