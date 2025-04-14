# Locations of the dependencies
KALDI_ROOT?=$(HOME)/travis/kaldi
OPENFST_ROOT?=$(KALDI_ROOT)/tools/openfst
OPENBLAS_ROOT?=$(KALDI_ROOT)/tools/OpenBLAS/install
MKL_ROOT?=/opt/intel/mkl
CUDA_ROOT?=/usr/local/cuda
USE_SHARED?=0
# Math libraries
HAVE_OPENBLAS_CLAPACK?=1
HAVE_MKL?=0
HAVE_ACCELERATE=0
HAVE_CUDA?=0
# Compiler
CXX?=g++
CC?=gcc
AR?=ar
STRIP?=strip
EXT?=so
# Extra
EXTRA_CFLAGS?=
EXTRA_LDFLAGS?=
OUTDIR?=.

VOSK_SOURCES= \
	recognizer.cc \
	language_model.cc \
	model.cc \
	spk_model.cc \
	vosk_api.cc \
	postprocessor.cc

VOSK_HEADERS= \
	recognizer.h \
	language_model.h \
	model.h \
	spk_model.h \
	vosk_api.h \
        postprocessor.h
# -g 为调试符号
CFLAGS= -g -O3 -std=c++17 -Wno-deprecated-declarations -fPIC -DFST_NO_DYNAMIC_LINKING -I. -I$(KALDI_ROOT)/src -I$(OPENFST_ROOT)/include $(EXTRA_CFLAGS)

OBJECTS = $(patsubst %.cc,$(OUTDIR)/%.o,$(VOSK_SOURCES))


LDFLAGS= -lpthread -ldl -lm -lrt -Wl,-rpath,$(OUTDIR)
ifeq ($(USE_SHARED), 0)
LDFLAGS += -Wl,--whole-archive
endif
ifeq ($(USE_SHARED), 0)
    LIBS = \
        $(KALDI_ROOT)/src/chain/kaldi-chain.a \
        $(KALDI_ROOT)/src/nnet2/kaldi-nnet2.a \
        $(KALDI_ROOT)/src/online2/kaldi-online2.a \
        $(KALDI_ROOT)/src/decoder/kaldi-decoder.a \
        $(KALDI_ROOT)/src/ivector/kaldi-ivector.a \
        $(KALDI_ROOT)/src/gmm/kaldi-gmm.a \
        $(KALDI_ROOT)/src/tree/kaldi-tree.a \
        $(KALDI_ROOT)/src/feat/kaldi-feat.a \
        $(KALDI_ROOT)/src/lat/kaldi-lat.a \
        $(KALDI_ROOT)/src/lm/kaldi-lm.a \
        $(KALDI_ROOT)/src/rnnlm/kaldi-rnnlm.a \
        $(KALDI_ROOT)/src/hmm/kaldi-hmm.a \
        $(KALDI_ROOT)/src/nnet3/kaldi-nnet3.a \
        $(KALDI_ROOT)/src/transform/kaldi-transform.a \
        $(KALDI_ROOT)/src/cudamatrix/kaldi-cudamatrix.a \
        $(KALDI_ROOT)/src/matrix/kaldi-matrix.a \
        $(KALDI_ROOT)/src/fstext/kaldi-fstext.a \
        $(KALDI_ROOT)/src/util/kaldi-util.a \
        $(KALDI_ROOT)/src/base/kaldi-base.a \
        $(OPENFST_ROOT)/lib/libfst.a \
        $(OPENFST_ROOT)/lib/libfstngram.a

    CFLAGS += -I$(KALDI_ROOT)/src \
            -I$(KALDI_ROOT)/src/chain/ \
            -I$(KALDI_ROOT)/src/nnet2/ \
            -I$(KALDI_ROOT)/src/online2/ \
            -I$(KALDI_ROOT)/src/decoder/ \
            -I$(KALDI_ROOT)/src/ivector/ \
            -I$(KALDI_ROOT)/src/gmm/ \
            -I$(KALDI_ROOT)/src/tree/ \
            -I$(KALDI_ROOT)/src/feat/ \
            -I$(KALDI_ROOT)/src/lat/ \
            -I$(KALDI_ROOT)/src/lm/ \
            -I$(KALDI_ROOT)/src/rnnlm/ \
            -I$(KALDI_ROOT)/src/hmm/ \
            -I$(KALDI_ROOT)/src/nnet3/ \
            -I$(KALDI_ROOT)/src/transform/ \
            -I$(KALDI_ROOT)/src/cudamatrix/ \
            -I$(KALDI_ROOT)/src/matrix/ \
            -I$(KALDI_ROOT)/src/fstext/ \
            -I$(KALDI_ROOT)/src/util/ \
            -I$(KALDI_ROOT)/src/base/
    LDFLAGS += \
        -lchain -lnnet2 -lonline2 -ldecoder -livector -lgmm -ltree \
        -lfeat -llat -llm -lrnnlm -lhmm -lnnet3 \
        -ltransform -lcudamatrix -lmatrix -lfstext \
        -lutil -lbase -lfst -lfstngram
else
    LDFLAGS += \
        -L$(KALDI_ROOT)/libs \
        -lkaldi-online2 -lkaldi-decoder -lkaldi-ivector -lkaldi-gmm -lkaldi-tree \
        -lkaldi-feat -lkaldi-lat -lkaldi-lm -lkaldi-rnnlm -lkaldi-hmm -lkaldi-nnet3 \
        -lkaldi-transform -lkaldi-cudamatrix -lkaldi-matrix -lkaldi-fstext \
        -lkaldi-util -lkaldi-base -lfst -lfstngram
endif
#-lf2c -llapack -lblas
ifeq ($(HAVE_OPENBLAS_CLAPACK), 1)
    CFLAGS += -I$(OPENBLAS_ROOT)/include
    ifeq ($(USE_SHARED), 0)
        LIBS += \
            $(OPENBLAS_ROOT)/lib/libopenblas.a \
            $(OPENBLAS_ROOT)/lib/liblapack.a \
            $(OPENBLAS_ROOT)/lib/libblas.a \
            $(OPENBLAS_ROOT)/lib/libf2c.a
       LDFLAGS += \
           -lf2c   -lblas  -llapack  -lopenblas
    else
        LDFLAGS += -lopenblas -llapack -lblas -lf2c
    endif
endif

ifeq ($(HAVE_MKL), 1)
    CFLAGS += -DHAVE_MKL=1 -I$(MKL_ROOT)/include
    LDFLAGS += -L$(MKL_ROOT)/lib/intel64 -Wl,-rpath=$(MKL_ROOT)/lib/intel64 -lmkl_rt -lmkl_intel_lp64 -lmkl_core -lmkl_sequential
endif

ifeq ($(HAVE_ACCELERATE), 1)
    LDFLAGS += -framework Accelerate
endif

ifeq ($(HAVE_CUDA), 1)
    VOSK_SOURCES += batch_recognizer.cc batch_model.cc
    VOSK_HEADERS += batch_recognizer.h batch_model.h

    CFLAGS+=-DHAVE_CUDA=1 -I$(CUDA_ROOT)/include

    LIBS := \
        $(KALDI_ROOT)/src/cudadecoder/kaldi-cudadecoder.a \
        $(KALDI_ROOT)/src/cudafeat/kaldi-cudafeat.a \
        $(LIBS)

    LDFLAGS += -L$(CUDA_ROOT)/lib64 -lcuda -lcublas -lcusparse -lcudart -lcurand -lcufft -lcusolver -lnvToolsExt
endif

ifeq ($(USE_SHARED), 0)
LDFLAGS += -Wl,--no-whole-archive
endif

LDFLAGS +=  -Wl,--strip-debug

# 添加ARMv7架构指定
CFLAGS += -march=armv7-a -mfpu=neon-vfpv4 -mfloat-abi=hard


# 默认目标
all: $(OUTDIR)/libvosk.$(EXT)

$(OUTDIR)/libvosk.$(EXT): $(OBJECTS)
	@find $(OUTDIR) -name '*.o' -exec $(OBJCOPY) --strip-symbol=lsame_ {} \;

	@echo "=> 解压必要的库..."
	@mkdir  -p  "$(OUTDIR)/libs"
	@cur_dir=`pwd` && \
	for lib in $(LIBS); do \
		if [ -f "$$lib" ]; then \
			cd "$$cur_dir" && \
			libname=`basename $$lib | sed 's/\.a$$//'` && \
			echo "处理库: $$libname (路径: $$lib)" && \
			mkdir -p "$(OUTDIR)/tmp/$$libname" && \
			newlibname=`echo $$libname | sed 's/^kaldi-/lib/'` && \
            cp "$$lib" "$(OUTDIR)/libs/$$newlibname.a" && \
			(cd "$(OUTDIR)/tmp/$$libname" && $(AR) x "$$lib" && \
			for f in *.o; do mv "$$f" "$${libname}_$$f"; done) ; \
		else \
			echo "错误: 找不到库文件 $$lib" && \
			exit 1 ; \
		fi ; \
	done

	@echo "=> 移除冲突符号..."
	@find "$(OUTDIR)/tmp/libblas" -name 'libblas_lsame*.o' -exec sh -c '$(OBJCOPY)  --strip-symbol=lsame_  "{}"' \;
	@find "$(OUTDIR)/tmp/libopenblas" -name '*.o' -exec sh -c '$(OBJCOPY)  --strip-symbol=lsame_  --strip-symbol=c_abs --strip-symbol=dcabs1_ --strip-symbol=scabs1_ --strip-symbol=z_abs "{}"' \;
	@find "$(OUTDIR)/tmp/libf2c" -name '*.o' -exec sh -c '$(OBJCOPY) --strip-symbol=main "{}"' \;


	@echo "=> 处理缺少的.note.GNU-stack段..."
	@echo "" | $(AS) -o temp_stack.o -c -   # 生成临时汇编目标文件
	@for obj in $$(find "$(OUTDIR)/tmp" -name "*.o"); do \
    if ! $(OBJDUMP) -h "$$obj" 2>/dev/null | grep -q '.note.GNU-stack'; then \
        $(OBJCOPY) --add-section .note.GNU-stack=temp_stack.o "$$obj"; \
    fi; \
	done
	@rm -f temp_stack.o

	@echo "=> 将所有处理好的.o文件移动到$(OUTDIR)/tmp"
	@find $(OUTDIR)/tmp -name '*.o' -exec mv {} $(OUTDIR)/tmp \;
	@$(CC) -c -march=armv7-a -mfpu=neon -mfloat-abi=hard  dummy_main.c -o $(OUTDIR)/dummy_main.o
	$(AR) rcs  $@  $(OBJECTS)  $(OUTDIR)/dummy_main.o  $$(ls $(OUTDIR)/tmp/*.o 2>/dev/null)
	# 然后链接
	# @$(CXX) $(CFLAGS) -static-libstdc++ -static-libgcc -u MAIN__ -Wl,--allow-multiple-definition -o  $@  -L$(OUTDIR)/libs -Wl,--exclude-libs,liblapack.a:libblas.a:libf2c.a $(LDFLAGS) $(EXTRA_LDFLAGS)

	@$(STRIP) --strip-debug $@
	@echo "=> 处理库: libvosk.$(EXT) (路径: $@)"


	@echo "处理完成. 请检查新库是否满足预期."
	@echo "构建成功! 输出文件: $@"
	@echo "优化后大小: $$(du -h $@ | cut -f1)"
	@echo "=> 清理临时文件..."
	@rm -rf $(OUTDIR)/tmp  $(OUTDIR)/*.o
	@mkdir -p $(OUTDIR)/tmp

#	@echo "=> 启动静态库极致瘦身流程"
#	# 备份原始文件
#	@cp $@ $@.bak 2>/dev/null || true
#	# 阶段1：解包并处理目标文件
#	@echo "解包并处理目标文件..."
#	@cd $(OUTDIR)/tmp && \
#	$(AR) x ../$(notdir $@) 2>/dev/null || true && \
#    find . -name '*.o' -exec $(OBJCOPY) \
#        --compress-debug-sections=zlib \
#        --strip-symbol=.L* \
#        --remove-section=.comment \
#        --remove-section=.note* {} \;
#
#	# 阶段2：瘦身后重新打包
#	@echo "重建Thin Archive..."
#	@$(AR) crs --thin $@.2 $(OUTDIR)/tmp/*.o
#
#	# 精准文件大小统计（兼容交叉编译环境）
#	@echo "优化后大小: $$(stat -f '%z' $@ | numfmt --to=iec --suffix=B --format='%.2f')"
#	@echo "优化后大小: $$(stat -f '%z' $@.2 | numfmt --to=iec --suffix=B --format='%.2f')"
#	@echo "处理完成. 请检查新库是否满足预期."
#	@echo "构建成功! 输出文件: $@"

	@rm -rf $(OUTDIR)/tmp

# 编译源文件为目标文件
$(OUTDIR)/%.o: %.cc $(VOSK_HEADERS)
	@mkdir -p $(OUTDIR)
	$(CXX) $(CFLAGS) -c -o $@  $<

clean:
	rm -f *.o *.so *.dll
