# Copyright 2016 Google Inc.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#
################################################################################

# Using multi-stage build to copy some LLVM binaries needed in the runner image.
FROM gcr.io/oss-fuzz-base/base-clang AS base-clang

FROM gcr.io/oss-fuzz-base/base-image

# Copy the binaries needed for code coverage and crash symbolization.
COPY --from=base-clang /usr/local/bin/llvm-cov /usr/local/bin/
COPY --from=base-clang /usr/local/bin/llvm-profdata /usr/local/bin/
COPY --from=base-clang /usr/local/bin/llvm-symbolizer /usr/local/bin/

RUN apt-get update
RUN apt-get install -y \
    binutils \
    file \
    fonts-dejavu \
    git \
    libc6-dev-i386 \
    libcap2 \
    python3 \
    python3-pip \
    wget \
    zip

RUN git clone https://chromium.googlesource.com/chromium/src/tools/code_coverage /opt/code_coverage
RUN pip3 install -r /opt/code_coverage/requirements.txt

COPY bad_build_check \
    collect_dft \
    coverage \
    coverage_helper \
    dataflow_tracer.py \
    download_corpus \
    minijail0 \
    reproduce \
    run_fuzzer \
    run_minijail \
    parse_options.py \
    targets_list \
    test_all \
    test_one \
    /usr/local/bin/

# Default environment options for various sanitizers.
# Note that these match the settings used in ClusterFuzz and
# shouldn't be changed unless a corresponding change is made on
# ClusterFuzz side as well.
ENV ASAN_OPTIONS="alloc_dealloc_mismatch=0:allocator_may_return_null=1:allocator_release_to_os_interval_ms=500:check_malloc_usable_size=0:detect_container_overflow=1:detect_odr_violation=0:detect_leaks=1:detect_stack_use_after_return=1:fast_unwind_on_fatal=0:handle_abort=1:handle_segv=1:handle_sigill=1:max_uar_stack_size_log=16:print_scariness=1:quarantine_size_mb=10:strict_memcmp=1:strip_path_prefix=/workspace/:symbolize=1:use_sigaltstack=1:dedup_token_length=3"
ENV MSAN_OPTIONS="print_stats=1:strip_path_prefix=/workspace/:symbolize=1:dedup_token_length=3"
ENV UBSAN_OPTIONS="print_stacktrace=1:print_summary=1:silence_unsigned_overflow=1:strip_path_prefix=/workspace/:symbolize=1:dedup_token_length=3"
ENV FUZZER_ARGS="-rss_limit_mb=2560 -timeout=25"
ENV AFL_FUZZER_ARGS="-m none"

# Download and install the latest stable Go.
ADD https://storage.googleapis.com/golang/getgo/installer_linux $SRC/
RUN chmod +x $SRC/installer_linux && \
    SHELL="bash" $SRC/installer_linux && \
    rm $SRC/installer_linux

# Set up Golang environment variables (copied from /root/.bash_profile).
ENV GOPATH /root/go

# /root/.go/bin is for the standard Go binaries (i.e. go, gofmt, etc).
# $GOPATH/bin is for the binaries from the dependencies installed via "go get".
ENV PATH $PATH:/root/.go/bin:$GOPATH/bin

# gocovmerge merges coverage profiles.
RUN go get -u github.com/wadey/gocovmerge
# pprof-merge merges performance profiles.
RUN go get -u github.com/rakyll/pprof-merge
# gocovsum produces a json summary.
RUN go get -u github.com/catenacyber/gocovsum

