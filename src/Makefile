ifeq ($(TARGET),ARMV7)    # Usage: 'make android TARGET=ARMV7'
	THE_OS=Android
	SATC=$(HOME)/android/android-ndk-r17c/satl23
	CXX=arm-linux-androideabi-clang++
	CXXFLAGS += -march=armv7
	LDFLAGS += -march=armv7
	BOOST_ARCH=armeabi-v7a
else
	THE_OS := $(shell uname -s)
endif

default:
	@echo "Detected OS: ${THE_OS}"
	$(MAKE) CC=gcc CXX=g++ \
		CXXFLAGS='$(CXXFLAGS) -Wall -Wextra -Wno-ignored-attributes -pipe -O3 -g -ffast-math -flto -march=native -std=c++14 -DNDEBUG'  \
		LDFLAGS='$(LDFLAGS) -flto -g' \
		leelaz

debug:
	@echo "Detected OS: ${THE_OS}"
	$(MAKE) CC=gcc CXX=g++ \
		CXXFLAGS='$(CXXFLAGS) -Wall -Wextra -Wno-ignored-attributes -pipe -Og -g -std=c++14' \
		LDFLAGS='$(LDFLAGS) -g' \
		leelaz

clang:
	@echo "Detected OS: ${THE_OS}"
	$(MAKE) CC=clang CXX=clang++ \
		CXXFLAGS='$(CXXFLAGS) -Wall -Wextra -Wno-missing-braces -Wno-mismatched-tags -O3 -ffast-math -flto -march=native -std=c++14 -DNDEBUG' \
		LDFLAGS='$(LDFLAGS) -flto -fuse-linker-plugin' \
		leelaz

ifeq ($(THE_OS),Android)
export PATH := $(SATC)/bin:$(PATH)
android:
	@echo "Detected OS: ${THE_OS}"
	$(MAKE) CXXFLAGS='$(CXXFLAGS) -Wall -Wextra -Wno-missing-braces -O3 -ffast-math -DNDEBUG' \
		LDFLAGS='$(LDFLAGS)  -flto -fuse-linker-plugin' \
		leelaz
endif

DYNAMIC_LIBS = 
LIBS =

ifeq ($(THE_OS),Android)
	# Extra flags
	CXXFLAGS += -std=c++14 -fPIE -D__ANDROID_API__=23 -Wno-macro-redefined -DUSE_CPU_ONLY
	LDFLAGS += -fPIE -pie -D__ANDROID_API__=23
	# search local path for shared libraries?
	#LDFLAGS += -Wl,-rpath,.

    # Eigen
    CXXFLAGS += -I./Eigen

	# Static?
	#CXXFLAGS += -static
	LDFLAGS += -static-libstdc++

	# Boost for android
	CXXFLAGS += -I$(SATC)/extra/boost/1.68.0/include
	LDFLAGS += -L$(SATC)/extra/boost/1.68.0/libs/llvm/$(BOOST_ARCH)
	#LIBS += $(SATC)/extra/boost/1.68.0/libs/llvm/$(BOOST_ARCH)/libboost_program_options.a
	DYNAMIC_LIBS += -lboost_program_options -lboost_filesystem -lboost_system

	# OpenBlass for android
	CXXFLAGS += -I$(SATC)/extra/openblas/include
	LDFLAGS += -L$(SATC)/extra/openblas/lib
	LIBS += $(SATC)/extra/openblas/lib/libopenblas.a
	#DYNAMIC_LIBS += -lopenblas 

	# libz
	#LIBS += $(SATC)/sysroot/usr/lib/libz.a
	DYNAMIC_LIBS += -lz
else 
	DYNAMIC_LIBS = -lboost_filesystem -lboost_system -lboost_program_options -lpthread -lz
endif

ifeq ($(THE_OS),Linux)
# for Linux with OpenBLAS
	CXXFLAGS += -I/usr/include/openblas -I./Eigen
	DYNAMIC_LIBS += -lopenblas
	DYNAMIC_LIBS += -lOpenCL
endif
ifeq ($(THE_OS),Darwin)
# for macOS (comment out the Linux part)
	LIBS += -framework Accelerate
	LIBS += -framework OpenCL
	CXXFLAGS += -I./Eigen
	CXXFLAGS += -I/System/Library/Frameworks/Accelerate.framework/Versions/Current/Headers
endif

# for MKL instead of OpenBLAS
#DYNAMIC_LIBS += -lmkl_rt
#CXXFLAGS += -I/opt/intel/mkl/include
#LDFLAGS  += -L/opt/intel/mkl/lib/intel64/

CXXFLAGS += -I.
CPPFLAGS += -MD -MP

# When compiling for Android -flto does not work with Random.cpp (static thread_local), but for other objects it works fine
special_sources = Random.cpp
sources = Network.cpp FullBoard.cpp KoState.cpp Training.cpp \
	  TimeControl.cpp UCTSearch.cpp GameState.cpp Leela.cpp \
	  SGFParser.cpp Timing.cpp Utils.cpp FastBoard.cpp \
	  SGFTree.cpp Zobrist.cpp FastState.cpp GTP.cpp \
	  SMP.cpp UCTNode.cpp UCTNodePointer.cpp UCTNodeRoot.cpp \
	  OpenCL.cpp OpenCLScheduler.cpp NNCache.cpp Tuner.cpp CPUPipe.cpp

special_objects = $(special_sources:.cpp=.o)
objects = $(sources:.cpp=.o)

all_sources = $(special_sources) $(sources)
deps = $(all_sources:%.cpp=%.d)
-include $(deps)

ifeq ($(THE_OS),Android)
$(objects): EXTRA_FLAGS := -flto # Link Time Optimization for normal objects, excluding Random
endif

%.o: %.cpp
	@echo [Compiling]: $<
	$(CXX) $(CXXFLAGS) $(CPPFLAGS) $(EXTRA_FLAGS) -c -o $@ $<

leelaz: $(special_objects) $(objects)
	@echo [Linking]
	$(CXX) $(LDFLAGS) -o $@ $^ $(LIBS) $(DYNAMIC_LIBS)

clean:
	-$(RM) leelaz $(objects) $(special_objects) $(deps)

.PHONY: clean default debug clang
