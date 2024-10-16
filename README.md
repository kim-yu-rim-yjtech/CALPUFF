# WRF 모델 설치
## WRF 기본 설정
### wrf 설치를 위한 기본 라이브러리 설치
- sudo apt update
- sudo apt upgrade

### 라이브러리 설치
- sudo apt install gcc-9 gfortran-9 g++-9 libtool automake autoconf make m4 grads default-jre csh
- sudo apt-get install libtirpc-dev
- export CFLAGS="-I/usr/include/tirpc"
- export LDFLAGS="-L/usr/lib/x86_64-linux-gnu"
        - Error 발생 시
            - sudo add-apt-repository ppa:ubuntu-toolchain-r/test
            - sudo apt-get update
            - sudo apt-get install gfortran-9
            - sudo update-alternatives --install /usr/bin/gfortran gfortran /usr/bin/gfortran-9 100
    - gfortran --version
- sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-9 90
- sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-9 90
- sudo update-alternatives --install /usr/bin/gfortran gfortran /usr/bin/gfortran-9 90


### 환경 변수 설정
- export HOME='/home/yurim' <b> &nbsp; &nbsp; &nbsp; &nbsp; ※ home 뒤 폴더는 본인 경로로</b>
- mkdir $HOME/CALPUFF/WRF
- cd WRF
- mkdir Downloads <b> &nbsp; &nbsp; &nbsp; &nbsp; ※ wget으로 다운로드 받는 파일들을 모아둘 디렉토리</b>
- mkdir Library <b> &nbsp; &nbsp; &nbsp; &nbsp; ※ 많은 라이브러리들을 실질적으로 설치할 디렉토리</b>

### Downloads 내 다운로드
- cd Downloads
- wget -c https://www.zlib.net/fossils/zlib-1.2.13.tar.gz
- wget -c https://docs.hdfgroup.org/archive/support/ftp/HDF5/releases/hdf5-1.10/hdf5-1.10.5/src/hdf5-1.10.5.tar.gz
- wget -c https://downloads.unidata.ucar.edu/netcdf-c/4.9.0/netcdf-c-4.9.0.tar.gz
- wget -c https://downloads.unidata.ucar.edu/netcdf-fortran/4.6.0/netcdf-fortran-4.6.0.tar.gz
- wget -c http://www.mpich.org/static/downloads/3.3.1/mpich-3.3.1.tar.gz
- wget -c https://download.sourceforge.net/libpng/libpng-1.6.37.tar.gz
- wget -c https://www.ece.uvic.ca/~frodo/jasper/software/jasper-1.900.1.zip

### 환경 변수 설정
- export DIR=$HOME/CALPUFF/WRF/Library
- export CC=gcc
- export CXX=g++
- export FC=gfortran
- export F77=gfortran


### zlib 설치
- tar -xvzf zlib-1.2.13.tar.gz
- cd zlib-1.2.13/
- ./configure --prefix=$DIR
- make
- make install


### HDF5 설치
- cd ../
- tar -xvzf hdf5-1.10.5.tar.gz
- cd hdf5-1.10.5/
- ./configure --prefix=$DIR --with-zlib=$DIR --enable-hl --enable-fortran
- make check >& make.log
- make install

#### 환경 변수 설정
- export HDF5=$DIR
- export LD_LIBRARY_PATH=$DIR/lib:$LD_LIBRARY_PATH


### netCDF - c 설치
- cd ../
- tar xvzf netcdf-c-4.9.0.tar.gz
- cd netcdf-c-4.9.0
- export CPPFLAGS=-I$DIR/include 
- export LDFLAGS=-L$DIR/lib
- ./configure --prefix=$DIR --disable-dap
- make check >& make.log
- make install

#### 환경 변수 설정
- export PATH=$DIR/bin:$PATH
- export NETCDF=$DIR


### netCDF - fortran 설치
- cd ../
- tar -xvzf netcdf-fortran-4.6.0.tar.gz
- cd netcdf-fortran-4.6.0/
- export LD_LIBRARY_PATH=$DIR/lib:$LD_LIBRARY_PATH
- export CPPFLAGS=-I$DIR/include 
- export LDFLAGS=-L$DIR/lib
- export LIBS="-lnetcdf -lhdf5_hl -lhdf5 -lz" 
- ./configure --prefix=$DIR --disable-shared
- make check >& make.log
- make install


### MPICH 설치
- cd ../
- tar -xvzf mpich-3.3.1.tar.gz
- cd mpich-3.3.1/
- ./configure --prefix=$DIR --enable-fortran
- make >& make.log  
- make install


### libpng 설치
- cd ../ 
- export LDFLAGS=-L$DIR/lib
- export CPPFLAGS=-I$DIR/include
- tar -xvzf libpng-1.6.37.tar.gz
- cd libpng-1.6.37/
- ./configure --prefix=$DIR
- make >& make.log
- make install


### jasper 설치
- cd ../
- sudo apt install unzip
- sudo apt update
- unzip jasper-1.900.1.zip
- cd jasper-1.900.1/
- autoreconf -i
- ./configure --prefix=$DIR
- make >& make.log
- make install

#### 환경 변수 설정
- export JASPERLIB=$DIR/lib
- export JASPERINC=$DIR/include

## WRF 모델 설치
- cd ../
- tar -xvzf /home/yurim/CALPUFF/WRF/Downloads/WRF-3.9.1.tar.gz -C $HOME/CALPUFF/WRF
- cd ../WRF-3.9.1
- ./clean
- ./configure => 34번 선택, 1번 선택
- export CPPFLAGS=-I$DIR/include/tirpc
- ./compile em_real >& compile.log 
        - ls /home/yurim/CALPUFF/WRF/Library/lib해서 libnetcdff.a, libnetcdf.a, libhdf5_fortran.a, libhdf5.a, libz.a들이 있는지 확인
        - sudo apt-get install libnetcdf-c++4 libnetcdf-c++4-dev
        - congifure.wrf 수정
            - LDFLAGS         =    $(OMP) $(FCFLAGS) $(LDFLAGS_LOCAL) -ltirpc

            - CFLAGS          =    $(CFLAGS_LOCAL) -DDM_PARALLEL  \
                      -DMAX_HISTORY=$(MAX_HISTORY) -DNMM_CORE=$(WRF_NMM_CORE) \
                      -I/usr/include/tirpc
            - LIB_EXTERNAL    = \
                      -L$(WRF_SRC_ROOT_DIR)/external/io_netcdf -lwrfio_nf -L/home/yurim/CALPUFF/WRF/Library/lib -lnetcdff -lnetcdf     -L/home/yurim/CALPUFF/WRF/Library/lib -lhdf5_fortran -lhdf5 -lm -lz -ltirpc


- export WRF_DIR=$HOME/CALPUFF/WRF/WRF-3.9.1

## WPS 모델 설치
- cd ../
- tar -xvzf /home/yurim/CALPUFF/WRF/Downloads/WPSV3.9.1.TAR.gz -C $HOME/CALPUFF/WRF
- cd WPS
- ./configure => 3번 선택
- ./compile >& compile.log
        - configure.wps 열고 'DM_FC = mpif90 -f90=gfortran' 에서 '-f90=gfotran' 삭제
        - intmath.f에서 172줄 i-1_2, 207줄 i-1_1 변경


## WRF 실행하기
### 데이터 준비
- cd ../
- mkdir Data
- cd Data
- sudo apt update
- sudo apt install csh
- chmod +x rda-download.csh => 데이터 다운로드하기(https://rda.ucar.edu/datasets/d083002/dataaccess/)
- ./rda-download.csh

- cd ../
- mkdir WPS_GEOG
- cd WPS_GEOG 
- https://www2.mmm.ucar.edu/wrf/users/download/get_sources_wps_geog.html#mandatory 데이터 다운로드하기
- tar xvzf geog_high_res_mandatory.tar.gz => 이름 변경

### WPS 실행
- cd ../WPS
- ./link_grib.csh ../Data/fnl
- ln -sf ungrib/Variable_Tables/Vtable.GFS Vtable
- namelist.wps 수정
        - start_date
        - end_date
        - interval_seconds => 가져온 데이터의 시간 텀 
        - e_we
        - e_sn
        - geo_data_res
        - map_proj
        - dx => 3000: (3km), 1000: (1km) 
        - dy
        - geog_data_path => WPS_GEOG 폴더 경로로 변경 
        
    - 선택 수정
        - ref_lat
        - ref_lon
        - truelat1
        - truelat2
        - geog_data_res

- ./geogrid.exe 
- ./ungrib.exe 
- ./metgrid.exe
    - 에러 발생 시 환경 변수 확인
    - export LD_LIBRARY_PATH=/home/yurim/CALPUFF/WRF/Library/lib:$LD_LIBRARY_PATH
    - ldd ./metgrid.exe | grep netcdf => libnetcdf.so 파일이 출력 확인


### 결과 출력
- cd ../WRFV-3.9.1/test/em_real
- link_met_em.sh 파일 내 수정 =========== 파일 생성부터
- # WRF 모델 설치
## WRF 기본 설정
### wrf 설치를 위한 기본 라이브러리 설치
- sudo apt update
- sudo apt upgrade

### 라이브러리 설치
- sudo apt install gcc-9 gfortran-9 g++-9 libtool automake autoconf make m4 grads default-jre csh
- sudo apt-get install libtirpc-dev
- export CFLAGS="-I/usr/include/tirpc"
- export LDFLAGS="-L/usr/lib/x86_64-linux-gnu"
        - Error 발생 시
            - sudo add-apt-repository ppa:ubuntu-toolchain-r/test
            - sudo apt-get update
            - sudo apt-get install gfortran-9
            - sudo update-alternatives --install /usr/bin/gfortran gfortran /usr/bin/gfortran-9 100
    - gfortran --version
- sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-9 90
- sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-9 90
- sudo update-alternatives --install /usr/bin/gfortran gfortran /usr/bin/gfortran-9 90


### 환경 변수 설정
- export HOME='/home/yurim' <b> &nbsp; &nbsp; &nbsp; &nbsp; ※ home 뒤 폴더는 본인 경로로</b>
- mkdir $HOME/CALPUFF/WRF
- cd WRF
- mkdir Downloads <b> &nbsp; &nbsp; &nbsp; &nbsp; ※ wget으로 다운로드 받는 파일들을 모아둘 디렉토리</b>
- mkdir Library <b> &nbsp; &nbsp; &nbsp; &nbsp; ※ 많은 라이브러리들을 실질적으로 설치할 디렉토리</b>

### Downloads 내 다운로드
- cd Downloads
- wget -c https://www.zlib.net/fossils/zlib-1.2.13.tar.gz
- wget -c https://docs.hdfgroup.org/archive/support/ftp/HDF5/releases/hdf5-1.10/hdf5-1.10.5/src/hdf5-1.10.5.tar.gz
- wget -c https://downloads.unidata.ucar.edu/netcdf-c/4.9.0/netcdf-c-4.9.0.tar.gz
- wget -c https://downloads.unidata.ucar.edu/netcdf-fortran/4.6.0/netcdf-fortran-4.6.0.tar.gz
- wget -c http://www.mpich.org/static/downloads/3.3.1/mpich-3.3.1.tar.gz
- wget -c https://download.sourceforge.net/libpng/libpng-1.6.37.tar.gz
- wget -c https://www.ece.uvic.ca/~frodo/jasper/software/jasper-1.900.1.zip

### 환경 변수 설정
- export DIR=$HOME/CALPUFF/WRF/Library
- export CC=gcc
- export CXX=g++
- export FC=gfortran
- export F77=gfortran


### zlib 설치
- tar -xvzf zlib-1.2.13.tar.gz
- cd zlib-1.2.13/
- ./configure --prefix=$DIR
- make
- make install


### HDF5 설치
- cd ../
- tar -xvzf hdf5-1.10.5.tar.gz
- cd hdf5-1.10.5/
- ./configure --prefix=$DIR --with-zlib=$DIR --enable-hl --enable-fortran
- make check >& make.log
- make install

#### 환경 변수 설정
- export HDF5=$DIR
- export LD_LIBRARY_PATH=$DIR/lib:$LD_LIBRARY_PATH


### netCDF - c 설치
- cd ../
- tar xvzf netcdf-c-4.9.0.tar.gz
- cd netcdf-c-4.9.0
- export CPPFLAGS=-I$DIR/include 
- export LDFLAGS=-L$DIR/lib
- ./configure --prefix=$DIR --disable-dap
- make check >& make.log
- make install

#### 환경 변수 설정
- export PATH=$DIR/bin:$PATH
- export NETCDF=$DIR


### netCDF - fortran 설치
- cd ../
- tar -xvzf netcdf-fortran-4.6.0.tar.gz
- cd netcdf-fortran-4.6.0/
- export LD_LIBRARY_PATH=$DIR/lib:$LD_LIBRARY_PATH
- export CPPFLAGS=-I$DIR/include 
- export LDFLAGS=-L$DIR/lib
- export LIBS="-lnetcdf -lhdf5_hl -lhdf5 -lz" 
- ./configure --prefix=$DIR --disable-shared
- make check >& make.log
- make install


### MPICH 설치
- cd ../
- tar -xvzf mpich-3.3.1.tar.gz
- cd mpich-3.3.1/
- ./configure --prefix=$DIR --enable-fortran
- make >& make.log  
- make install


### libpng 설치
- cd ../ 
- export LDFLAGS=-L$DIR/lib
- export CPPFLAGS=-I$DIR/include
- tar -xvzf libpng-1.6.37.tar.gz
- cd libpng-1.6.37/
- ./configure --prefix=$DIR
- make >& make.log
- make install


### jasper 설치
- cd ../
- sudo apt install unzip
- sudo apt update
- unzip jasper-1.900.1.zip
- cd jasper-1.900.1/
- autoreconf -i
- ./configure --prefix=$DIR
- make >& make.log
- make install

#### 환경 변수 설정
- export JASPERLIB=$DIR/lib
- export JASPERINC=$DIR/include

## WRF 모델 설치
- cd ../
- tar -xvzf /home/yurim/CALPUFF/WRF/Downloads/WRF-3.9.1.tar.gz -C $HOME/CALPUFF/WRF
- cd ../WRF-3.9.1
- ./clean
- ./configure => 34번 선택, 1번 선택
- export CPPFLAGS=-I$DIR/include/tirpc
- ./compile em_real >& compile.log 
        - ls /home/yurim/CALPUFF/WRF/Library/lib해서 libnetcdff.a, libnetcdf.a, libhdf5_fortran.a, libhdf5.a, libz.a들이 있는지 확인
        - sudo apt-get install libnetcdf-c++4 libnetcdf-c++4-dev
        - congifure.wrf 수정
            - LDFLAGS         =    $(OMP) $(FCFLAGS) $(LDFLAGS_LOCAL) -ltirpc

            - CFLAGS          =    $(CFLAGS_LOCAL) -DDM_PARALLEL  \
                      -DMAX_HISTORY=$(MAX_HISTORY) -DNMM_CORE=$(WRF_NMM_CORE) \
                      -I/usr/include/tirpc
            - LIB_EXTERNAL    = \
                      -L$(WRF_SRC_ROOT_DIR)/external/io_netcdf -lwrfio_nf -L/home/yurim/CALPUFF/WRF/Library/lib -lnetcdff -lnetcdf     -L/home/yurim/CALPUFF/WRF/Library/lib -lhdf5_fortran -lhdf5 -lm -lz -ltirpc


- export WRF_DIR=$HOME/CALPUFF/WRF/WRF-3.9.1

## WPS 모델 설치
- cd ../
- tar -xvzf /home/yurim/CALPUFF/WRF/Downloads/WPSV3.9.1.TAR.gz -C $HOME/CALPUFF/WRF
- cd WPS
- ./configure => 3번 선택
- ./compile >& compile.log
        - configure.wps 열고 'DM_FC = mpif90 -f90=gfortran' 에서 '-f90=gfotran' 삭제
        - intmath.f에서 172줄 i-1_2, 207줄 i-1_1 변경


## WRF 실행하기
### 데이터 준비
- cd ../
- mkdir Data
- cd Data
- sudo apt update
- sudo apt install csh
- chmod +x rda-download.csh => 데이터 다운로드하기(https://rda.ucar.edu/datasets/d083002/dataaccess/)
- ./rda-download.csh

- cd ../
- mkdir WPS_GEOG
- cd WPS_GEOG 
- https://www2.mmm.ucar.edu/wrf/users/download/get_sources_wps_geog.html#mandatory 데이터 다운로드하기
- tar xvzf geog_high_res_mandatory.tar.gz => 이름 변경

### WPS 실행
- cd ../WPS
- ./link_grib.csh ../Data/fnl
- ln -sf ungrib/Variable_Tables/Vtable.GFS Vtable
- namelist.wps 수정
        - start_date
        - end_date
        - interval_seconds => 가져온 데이터의 시간 텀 
        - e_we
        - e_sn
        - geo_data_res
        - map_proj
        - dx => 3000: (3km), 1000: (1km) 
        - dy
        - geog_data_path => WPS_GEOG 폴더 경로로 변경 
        
    - 선택 수정
        - ref_lat
        - ref_lon
        - truelat1
        - truelat2
        - geog_data_res

- ./geogrid.exe 
- ./ungrib.exe 
- ./metgrid.exe
    - 에러 발생 시 환경 변수 확인
    - export LD_LIBRARY_PATH=/home/yurim/CALPUFF/WRF/Library/lib:$LD_LIBRARY_PATH
    - ldd ./metgrid.exe | grep netcdf => libnetcdf.so 파일이 출력 확인


### 결과 출력
- cd ../WRFV-3.9.1/test/em_real
- link_met_em.sh 파일 내 수정
- chmod +x /home/yurim/CALPUFF/WRF/WRFV3/test/em_real/link_met_em.sh => 해당 경로로 변경
- ./link_met_em.sh

- vi namelist.input 수정
        - run_days
        - run_hours
        - run_minutes
        - run_seconds
        - start_year
        - start_month
        - start_day
        - start_hour
        - end_year
        - end_month
        - end_day
        - end_hour
        - interval_seconds => link_met_em.sh 가져올 데이터의 시간 텀
        - history_interval => link_met_em.sh 출력할 데이터의 시간 텀
        - e_we
        - e_sn
        - dx
        - dy

- ./real.exe => 에러 발생시 rsl.error.0000 확인
- ls -ltrh
- ./wrf.exe => 오래 걸림 
- ls -ltrh



# CALWRF 모델 설치
 => 해당 경로로 변경
- ./link_met_em.sh

- vi namelist.input 수정
        - run_days
        - run_hours
        - run_minutes
        - run_seconds
        - start_year
        - start_month
        - start_day
        - start_hour
        - end_year
        - end_month
        - end_day
        - end_hour
        - interval_seconds => link_met_em.sh 가져올 데이터의 시간 텀
        - history_interval => link_met_em.sh 출력할 데이터의 시간 텀
        - e_we
        - e_sn
        - dx
        - dy

- ./real.exe => 에러 발생시 rsl.error.0000 확인
- ls -ltrh
- ./wrf.exe => 오래 걸림 
- ls -ltrh



# CALWRF 모델 설치
