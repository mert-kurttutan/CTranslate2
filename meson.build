project(
    'ctranslate2',
    'cpp',
    version: '1.1.0',
    subproject_dir: 'third_party',
)

add_project_arguments('-DCT2_X86_BUILD', language: 'cpp')

withmkl = get_option('with_mkl')

enable_cpu_dispatch = get_option('enable_cpu_dispatch')

if enable_cpu_dispatch
    message('Compiling for multiple CPU ISA and enabling runtime dispatch')
    add_project_arguments('-DCT2_WITH_CPU_DISPATCH', language: 'cpp')
endif


source = [
  'src/allocator.cc',
  'src/batch_reader.cc',
  'src/buffered_translation_wrapper.cc',
  'src/cpu/allocator.cc',
  'src/cpu/backend.cc',
  'src/cpu/cpu_info.cc',
  'src/cpu/cpu_isa.cc',
  'src/cpu/kernels.cc',
  'src/cpu/parallel.cc',
  'src/cpu/primitives.cc',
  'src/decoding.cc',
  'src/decoding_utils.cc',
  'src/devices.cc',
  'src/dtw.cc',
  'src/encoder.cc',
  'src/env.cc',
  'src/filesystem.cc',
  'src/generator.cc',
  'src/layers/attention_layer.cc',
  'src/layers/attention.cc',
  'src/layers/flash_attention.cc',
  'src/layers/common.cc',
  'src/layers/decoder.cc',
  'src/layers/transformer.cc',
  'src/layers/wav2vec2.cc',
  'src/layers/whisper.cc',
  'src/logging.cc',
  'src/models/language_model.cc',
  'src/models/model.cc',
  'src/models/model_factory.cc',
  'src/models/model_reader.cc',
  'src/models/sequence_to_sequence.cc',
  'src/models/transformer.cc',
  'src/models/wav2vec2.cc',
  'src/models/whisper.cc',
  'src/ops/activation.cc',
  'src/ops/add.cc',
  'src/ops/alibi_add.cc',
  'src/ops/alibi_add_cpu.cc',
  'src/ops/bias_add.cc',
  'src/ops/bias_add_cpu.cc',
  'src/ops/concat.cc',
  'src/ops/concat_split_slide_cpu.cc',
  'src/ops/conv1d.cc',
  'src/ops/conv1d_cpu.cc',
  'src/ops/cos.cc',
  'src/ops/dequantize.cc',
  'src/ops/dequantize_cpu.cc',
  'src/ops/flash_attention.cc',
  'src/ops/flash_attention_cpu.cc',
  'src/ops/gather.cc',
  'src/ops/gather_cpu.cc',
  'src/ops/gelu.cc',
  'src/ops/gemm.cc',
  'src/ops/gumbel_max.cc',
  'src/ops/gumbel_max_cpu.cc',
  'src/ops/layer_norm.cc',
  'src/ops/layer_norm_cpu.cc',
  'src/ops/log.cc',
  'src/ops/matmul.cc',
  'src/ops/mean.cc',
  'src/ops/mean_cpu.cc',
  'src/ops/median_filter.cc',
  'src/ops/min_max.cc',
  'src/ops/mul.cc',
  'src/ops/multinomial.cc',
  'src/ops/multinomial_cpu.cc',
  'src/ops/quantize.cc',
  'src/ops/quantize_cpu.cc',
  'src/ops/relu.cc',
  'src/ops/rms_norm.cc',
  'src/ops/rms_norm_cpu.cc',
  'src/ops/rotary.cc',
  'src/ops/rotary_cpu.cc',
  'src/ops/sin.cc',
  'src/ops/softmax.cc',
  'src/ops/softmax_cpu.cc',
  'src/ops/split.cc',
  'src/ops/slide.cc',
  'src/ops/sub.cc',
  'src/ops/swish.cc',
  'src/ops/tanh.cc',
  'src/ops/tile.cc',
  'src/ops/tile_cpu.cc',
  'src/ops/topk.cc',
  'src/ops/topk_cpu.cc',
  'src/ops/topp_mask.cc',
  'src/ops/topp_mask_cpu.cc',
  'src/ops/transpose.cc',
  'src/ops/nccl_ops.cc',
  'src/ops/nccl_ops_cpu.cc',
  'src/ops/awq/dequantize.cc',
  'src/ops/awq/dequantize_cpu.cc',
  'src/ops/awq/gemm.cc',
  'src/ops/awq/gemm_cpu.cc',
  'src/ops/awq/gemv.cc',
  'src/ops/awq/gemv_cpu.cc',
  'src/ops/sum.cc',
  'src/padder.cc',
  'src/profiler.cc',
  'src/random.cc',
  'src/sampling.cc',
  'src/scoring.cc',
  'src/storage_view.cc',
  'src/thread_pool.cc',
  'src/translator.cc',
  'src/types.cc',
  'src/utils.cc',
  'src/vocabulary.cc',
  'src/vocabulary_map.cc',
]

cmake = import('cmake')

opt_var_pic = cmake.subproject_options()

# Call CMake with `-DSOME_OTHER_VAR=ON`
opt_var_pic.add_cmake_defines({'CMAKE_POSITION_INDEPENDENT_CODE': true})

# Configure the CMake project
spd_proj = cmake.subproject('spdlog')

# Fetch the dependency object
spd_dep = spd_proj.dependency('spdlog_header_only')


# Configure the CMake project
cpu_proj = cmake.subproject('cpu_features', options: opt_var_pic)

# message('CMaket targets:\n - ' + '\n - '.join(cpu_proj.target_list()))

# Fetch the dependency object
cpu_dep = cpu_proj.dependency('cpu_features')



kernel_obj_arr = []

isa_to_flag = {
    'avx': ['-mavx'],
    'avx2': ['-mavx2', '-mfma'],
    'avx512': ['-mavx512f', '-mavx512bw', '-mavx512dq', '-mavx512vl', '-mavx512cd'],
}

# dispatch for each isa
foreach isa, flag : isa_to_flag
    kernel_obj = static_library(
        'cpu_kernel_' + isa,
        'src/cpu/kernels.cc',
        include_directories: ['include', 'src', 'third_party'],
        dependencies: [spd_dep, cpu_dep],
        cpp_args: flag,
        build_by_default: false,
    )

    kernel_obj_arr += kernel_obj
endforeach

library(
    'myctranslate', source,
    include_directories: ['include', 'src', 'third_party'],
    dependencies: [spd_dep, cpu_dep],
    link_whole: kernel_obj_arr,

)
