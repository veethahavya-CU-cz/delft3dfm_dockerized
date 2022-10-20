## setting up the system
# load the base image and update the system
FROM centos:7
SHELL ["/bin/bash", "-c"]

# update the system
#RUN cd /etc/yum.repos.d/ && sed -i 's/mirrorlist/#mirrorlist/g' /etc/yum.repos.d/CentOS-* && sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-*
RUN yum update -y

# install developement tools
RUN yum groupinstall "Development Tools" -y

# install dependencies from package manager - yum
RUN yum install -y m4 zlib zlib-devel ruby libuuid libuuid-devel uuid uuid-devel libcurl-devel openssl libtool libtool-ltdl-devel autoconf openssl-libs expat expat-devel autoconf libtool-ltdl libtool-ltdl-devel bison bison-devel flex flex-devel environment-modules wget glibc glibc-devel

# install intel oneapi
COPY oneAPI.repo /etc/yum.repos.d
RUN yum update -y
RUN yum install intel-hpckit -y

## setting up to compile and install dependencies
RUN mkdir -p /home/delft3dfm_compile/dependencies
WORKDIR /home/delft3dfm_compile/dependencies

# remove any old cmake versions and install the target version
RUN yum remove cmake -y
RUN wget https://github.com/Kitware/CMake/releases/download/v3.24.2/cmake-3.24.2-linux-x86_64.sh
RUN sh cmake-3.24.2-linux-x86_64.sh --prefix=/usr/local --skip-license

# install hdf5-1.10.6 with parallel io support
RUN wget github.com/HDFGroup/hdf5/archive/refs/tags/hdf5-1_10_6.tar.gz && tar -xf hdf5-1_10_6.tar.gz
WORKDIR /home/delft3dfm_compile/dependencies/hdf5-hdf5-1_10_6
RUN source /opt/intel/oneapi/setvars.sh && ./configure CC=mpiicc CXX=mpiicxx FC=mpiifort --enable-parallel --enable-shared --prefix=/usr/local CFLAGS="-m64 -diag-disable=10441"
RUN source /opt/intel/oneapi/setvars.sh && make install

# install netcdf c-libs with parallel io support
WORKDIR /home/delft3dfm_compile/dependencies
RUN wget github.com/Unidata/netcdf-c/archive/refs/tags/v4.6.1.tar.gz -O netcdf-c-4.6.1.tar.gz && tar -xf netcdf-c-4.6.1.tar.gz
WORKDIR /home/delft3dfm_compile/dependencies/netcdf-c-4.6.1
RUN source /opt/intel/oneapi/setvars.sh && ./configure CC=mpiicc CXX=mpiicxx FC=mpiifort --enable-parallel --enable-shared --prefix=/usr/local --enable-fortran --enable-remote-fortran-bootstrap --disable-dap-remote-tests CFLAGS="-m64 -diag-disable=10441"
RUN source /opt/intel/oneapi/setvars.sh && make install

## install netcdf fortran-libs
WORKDIR /home/delft3dfm_compile/dependencies
RUN wget github.com/Unidata/netcdf-fortran/archive/refs/tags/v4.5.0.tar.gz -O netcdf-fortran-4.5.0.tar.gz && tar -xf netcdf-fortran-4.5.0.tar.gz
WORKDIR /home/delft3dfm_compile/dependencies/netcdf-fortran-4.5.0
ENV LD_LIBRARY_PATH=/usr/local/lib:${LD_LIBRARY_PATH}
RUN source /opt/intel/oneapi/setvars.sh && ./configure CC=mpiicc CXX=mpiicxx FC=mpiifort F77=mpiifort --prefix=/usr/local --disable-fortran-type-check CPPFLAGS="-I/usr/local/include" LDFLAGS="-L/usr/local/lib" CFLAGS="-m64 -diag-disable=10441" --enable-shared --host=x86_64-pc-linux
RUN source /opt/intel/oneapi/setvars.sh && make install

# clean up src of dependencies
RUN rm -rf /home/delft3dfm_compile/dependencies


## compiling delft3dfm
# copy the zipped delft3dfm source code into the container
COPY delft3dfm_source/delft3dfm_68819.tar.gz /home/delft3dfm_compile/delft3dfm_68819.tar.gz
WORKDIR /home/delft3dfm_compile
RUN tar -xf delft3dfm_68819.tar.gz && rm delft3dfm_68819.tar.gz
WORKDIR /home/delft3dfm_compile/delft3dfm_68819/src/

# copy missing files: fix for known issue
RUN cp third_party_open/swan/src/*.[fF]* third_party_open/swan/swan_mpi && cp third_party_open/swan/src/*.[fF]* third_party_open/swan/swan_omp

# configure the build
RUN ./autogen.sh && source /opt/intel/oneapi/setvars.sh && ./configure CC=icc CXX=icpc FC=ifort MPICXX=mpiicpc F77=ifort MPIF77=mpiifort MPIFC=mpiifort CPPFLAGS="-I/usr/local/include" CFLAGS="-m64 -diag-disable=10441" NETCDF_CFLAGS="-I/usr/local/include" NETCDF_LIBS="-L/usr/local/lib -lnetcdf" --prefix=/opt/delft3dfm

# build
RUN source /etc/profile.d/modules.sh && source /opt/intel/oneapi/setvars.sh && CC=icc CXX=icpc FC=ifort MPICXX=mpiicpc F77=ifort MPIF77=mpiifort MPIFC=mpiifort CPPFLAGS="-I/usr/local/include" CFLAGS="-m64 -diag-disable=10441" make ds-install
RUN source /etc/profile.d/modules.sh && source /opt/intel/oneapi/setvars.sh && CC=icc CXX=icpc FC=ifort MPICXX=mpiicpc F77=ifort MPIF77=mpiifort MPIFC=mpiifort CPPFLAGS="-I/usr/local/include" CFLAGS="-m64 -diag-disable=10441" make ds-install -C engines_gpl/dflowfm


## delete source files and create a scratch dir for model runs
WORKDIR /home
RUN rm -rf delft3dfm_compile/ && mkdir scratch
WORKDIR /home/scratch