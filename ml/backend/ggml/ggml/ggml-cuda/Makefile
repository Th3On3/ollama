NVCC?=nvcc

CXX=$(NVCC)

CXXFLAGS= \
	-std=c++17 \
	$(foreach ARCHITECTURE,$(subst ;, ,$(CUDA_ARCHITECTURES)),--generate-code=arch=compute_$(ARCHITECTURE),code=sm_$(ARCHITECTURE)) \

CPPFLAGS= \
	-I.. \
	-I../include \

ALL_SOURCES=$(wildcard *.cu)
FATTN_SOURCES=$(wildcard fattn*.cu)

SOURCES= \
	$(filter-out $(FATTN_SOURCES),$(ALL_SOURCES)) \
	$(wildcard template-instances/mmq*.cu) \

ifneq ($(OLLAMA_FAST_BUILD),)
CPPFLAGS+=-DGGML_DISABLE_FLASH_ATTN
else
SOURCES+= \
	$(FATTN_SOURCES) \
	$(wildcard template-instances/fattn-wmma*.cu) \
	$(wildcard template-instances/fattn-vec*q4_0-q4_0.cu) \
	$(wildcard template-instances/fattn-vec*q8_0-q8_0.cu) \
	$(wildcard template-instances/fattn-vec*f16-f16.cu)
endif

all: cuda_v11 cuda_v12

cuda_v11: CUDA_ARCHITECTURES?=50;52;53;60;61;62;70;72;75;80;86
cuda_v11: OBJECTS=$(patsubst %.cu,%.o.v11,$(SOURCES))
cuda_v11: libggml_cuda_v11.a

cuda_v12: CUDA_ARCHITECTURES?=60;61;62;70;72;75;80;86;87;89;90;90a
cuda_v12: OBJECTS=$(patsubst %.cu,%.o.v12,$(SOURCES))
cuda_v12: libggml_cuda_v12.a

rocm: CPPFLAGS+=-DGGML_USE_HIP
rocm: OBJECTS=$(patsubst %.cu,%.o.rocm,$(SOURCES))
rocm: libggml_rocm.a

clean:
	$(RM) *.a *.o
	$(RM) template-instances/*.o

%.o.v11 %.o.v12 %.o.rocm: %.cu
	$(CXX) $(CXXFLAGS) $(CPPFLAGS) -c $< -o $@

.SECONDEXPANSION:
%.a: $$(OBJECTS)
	$(AR) rcs $@ $^
