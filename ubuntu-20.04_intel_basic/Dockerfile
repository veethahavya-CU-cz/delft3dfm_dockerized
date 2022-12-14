## setting up the system
# load the base image and update the system
FROM intel/oneapi-hpckit:devel-ubuntu20.04
SHELL ["/bin/bash", "-c"]

# update the system
RUN apt-get -y update && apt list --upgradable && apt-get -y upgrade

# install dependencies from package manager - apt
RUN apt-get install -y m4 zlib1g zlib1g-dev ruby uuid uuid-dev curl libcurl4 libcurl4-openssl-dev expat libexpat1-dev autoconf libtool bison flex environment-modules


## setting up to compile and install dependencies
RUN mkdir -p /home/delft3dfm_compile/dependencies
WORKDIR /home/delft3dfm_compile/dependencies

# install hdf5-1.10.6 with parallel io support
RUN wget github.com/HDFGroup/hdf5/archive/refs/tags/hdf5-1_10_6.tar.gz && tar -xf hdf5-1_10_6.tar.gz
WORKDIR /home/delft3dfm_compile/dependencies/hdf5-hdf5-1_10_6
RUN ./configure CC=mpiicc CXX=mpiicxx FC=mpiifort --enable-parallel --enable-shared --prefix=/usr/local CFLAGS="-m64 -diag-disable=10441"
RUN make install

# install netcdf c-libs with parallel io support
WORKDIR /home/delft3dfm_compile/dependencies
RUN wget github.com/Unidata/netcdf-c/archive/refs/tags/v4.6.1.tar.gz -O netcdf-c-4.6.1.tar.gz && tar -xf netcdf-c-4.6.1.tar.gz
WORKDIR /home/delft3dfm_compile/dependencies/netcdf-c-4.6.1
RUN ./configure CC=mpiicc CXX=mpiicxx FC=mpiifort --enable-parallel --enable-shared --prefix=/usr/local --enable-fortran --enable-remote-fortran-bootstrap --disable-dap-remote-tests CFLAGS="-m64 -diag-disable=10441"
RUN make install

## install netcdf fortran-libs
WORKDIR /home/delft3dfm_compile/dependencies
RUN wget github.com/Unidata/netcdf-fortran/archive/refs/tags/v4.5.0.tar.gz -O netcdf-fortran-4.5.0.tar.gz && tar -xf netcdf-fortran-4.5.0.tar.gz
WORKDIR /home/delft3dfm_compile/dependencies/netcdf-fortran-4.5.0
ENV LD_LIBRARY_PATH=/usr/local/lib:${LD_LIBRARY_PATH}
RUN ./configure CC=mpiicc CXX=mpiicxx FC=mpiifort F77=mpiifort --prefix=/usr/local --disable-fortran-type-check CPPFLAGS="-I/usr/local/include" LDFLAGS="-L/usr/local/lib" CFLAGS="-m64 -diag-disable=10441" --enable-shared --host=x86_64-pc-linux
RUN make install

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
RUN ./autogen.sh && ./configure CC=icc CXX=icpc FC=ifort MPICXX=mpiicpc F77=ifort MPIF77=mpiifort MPIFC=mpiifort CPPFLAGS="-I/usr/local/include" CFLAGS="-m64 -diag-disable=10441" NETCDF_CFLAGS="-I/usr/local/include" NETCDF_LIBS="-L/usr/local/lib -lnetcdf" --prefix=/opt/delft3dfm

# build
RUN source /etc/profile.d/modules.sh && CC=icc CXX=icpc FC=ifort MPICXX=mpiicpc F77=ifort MPIF77=mpiifort MPIFC=mpiifort CPPFLAGS="-I/usr/local/include" CFLAGS="-m64 -diag-disable=10441" make ds-install
RUN source /etc/profile.d/modules.sh && CC=icc CXX=icpc FC=ifort MPICXX=mpiicpc F77=ifort MPIF77=mpiifort MPIFC=mpiifort CPPFLAGS="-I/usr/local/include" CFLAGS="-m64 -diag-disable=10441" make ds-install -C engines_gpl/dflowfm


# create a scratch dir for model runs
RUN mkdir /home/scratch