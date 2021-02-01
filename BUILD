load("@rules_cc//cc:defs.bzl", "cc_binary", "cc_library")
load("@bazel_skylib//rules:run_binary.bzl", "run_binary")
load(
    "//:utils.bzl",
    "cmake_var_string",
    "expand_cmake_vars",
    "llvm_all_cmake_vars",
    "llvm_copts",
    "llvm_defines",
    "llvm_linkopts",
    "table_gen_library",
)
load("//:template_rule.bzl", "template_rule")
load("@rules_winsdk//:win_etw_library.bzl", "win_etw_library")
load("@rules_winsdk//:win_res_library.bzl", "win_res_library")

package(licenses = ["notice"])

exports_files(["LICENSE.TXT"])

# main target
cc_library(
    name = "directx_shader_compiler",
    hdrs = [
        "include/dxc/dxcapi.h",
    ],
    includes = [
        "include/dxc",
    ],
    visibility = ["//visibility:public"],
    deps = [
        ":dxcompiler",
    ],
)

# HLSL removed targets AArch64, ARM, BPF, CppBackend, Hexagon, Mips, MSP430, PowerPC, Sparc, SystemZ, X86, XCore
llvm_targets = [
    #"AMDGPU",
    #"NVPTX",
]

py_binary(
    name = "expand_cmake_vars",
    srcs = ["expand_cmake_vars.py"],
    srcs_version = "PY2AND3",
    visibility = ["@directx_shader_compiler//:__subpackages__"],
)

py_binary(
    name = "gen_version",
    srcs = ["utils/version/gen_version.py"],
    srcs_version = "PY2AND3",
    visibility = ["@directx_shader_compiler//:__subpackages__"],
)

llvm_target_asm_parsers = llvm_targets

llvm_target_asm_printers = llvm_targets

llvm_target_disassemblers = llvm_targets

# Performs CMake variable substitutions on configuration header files.
expand_cmake_vars(
    name = "clang_config_gen",
    src = "tools/clang/include/clang/Config/config.h.cmake",
    cmake_vars = cmake_var_string({
        "BUG_REPORT_URL": "http://llvm.org/bugs/",
        "CLANG_DEFAULT_OPENMP_RUNTIME": "libgomp",
        "CLANG_LIBDIR_SUFFIX": "",
        "CLANG_RESOURCE_DIR": "",
        "C_INCLUDE_DIRS": "",
        "DEFAULT_SYSROOT": "",
        "GCC_INSTALL_PREFIX": "",
        "BACKEND_PACKAGE_STRING": "LLVM 3.7-v1.5.2003",
    }),
    dst = "tools/clang/include/clang/Config/config.h",
)

expand_cmake_vars(
    name = "config_gen",
    src = "include/llvm/Config/config.h.cmake",
    cmake_vars = llvm_all_cmake_vars,
    dst = "include/llvm/Config/config.h",
)

expand_cmake_vars(
    name = "llvm_config_gen",
    src = "include/llvm/Config/llvm-config.h.cmake",
    cmake_vars = llvm_all_cmake_vars,
    dst = "include/llvm/Config/llvm-config.h",
)

expand_cmake_vars(
    name = "datatypes_gen",
    src = "include/llvm/Support/DataTypes.h.cmake",
    cmake_vars = llvm_all_cmake_vars,
    dst = "include/llvm/Support/DataTypes.h",
)

run_binary(
    name = "hlsl_dxcversion_autogen",
    srcs = [
        "utils/version/latest-release.json",
    ],
    outs = [
        "include/dxcversion.inc",
    ],
    args = [
        "--no-commit-sha",
        "--official",
        "--release-file",
        "$(location utils/version/latest-release.json)",
        "--out-file",
        "$(location include/dxcversion.inc)",
    ],
    tool = ":gen_version",
)

cc_library(
    name = "hlsl_dxcversion",
    textual_hdrs = [
        "include/dxcversion.inc",
    ],
)

# Performs macro expansions on .def.in files
template_rule(
    name = "targets_def_gen",
    src = "include/llvm/Config/Targets.def.in",
    out = "include/llvm/Config/Targets.def",
    substitutions = {
        "@LLVM_ENUM_TARGETS@": "\n".join(
            ["LLVM_TARGET({})".format(t) for t in llvm_targets],
        ),
    },
)

template_rule(
    name = "asm_parsers_def_gen",
    src = "include/llvm/Config/AsmParsers.def.in",
    out = "include/llvm/Config/AsmParsers.def",
    substitutions = {
        "@LLVM_ENUM_ASM_PARSERS@": "\n".join(
            ["LLVM_ASM_PARSER({})".format(t) for t in llvm_target_asm_parsers],
        ),
    },
)

template_rule(
    name = "asm_printers_def_gen",
    src = "include/llvm/Config/AsmPrinters.def.in",
    out = "include/llvm/Config/AsmPrinters.def",
    substitutions = {
        "@LLVM_ENUM_ASM_PRINTERS@": "\n".join(
            ["LLVM_ASM_PRINTER({})".format(t) for t in llvm_target_asm_printers],
        ),
    },
)

template_rule(
    name = "disassemblers_def_gen",
    src = "include/llvm/Config/Disassemblers.def.in",
    out = "include/llvm/Config/Disassemblers.def",
    substitutions = {
        "@LLVM_ENUM_DISASSEMBLERS@": "\n".join(
            ["LLVM_DISASSEMBLER({})".format(t) for t in llvm_target_disassemblers],
        ),
    },
)

template_rule(
    name = "clang_version",
    src = "tools/clang/include/clang/Basic/Version.inc.in",
    out = "tools/clang/include/clang/Basic/Version.inc",
    substitutions = {
        "@CLANG_VERSION@": "3.7",
        "@CLANG_VERSION_MAJOR@": "3",
        "@CLANG_VERSION_MINOR@": "7",
        "@CLANG_HAS_VERSION_PATCHLEVEL@": "0",
        "@CLANG_VERSION_PATCHLEVEL@": "",
    },
)

# A common library that all LLVM targets depend on.
cc_library(
    name = "config",
    hdrs = glob([
        "**/*.h",
        "**/*.def",
        "**/*.inc.cpp",
    ]) + [
        "include/llvm/Config/AsmParsers.def",
        "include/llvm/Config/AsmPrinters.def",
        "include/llvm/Config/Disassemblers.def",
        "include/llvm/Config/Targets.def",
        "include/llvm/Config/config.h",
        "include/llvm/Config/llvm-config.h",
    ],
    defines = llvm_defines,
    includes = ["include"],
)

cc_library(
    name = "clang_config",
    hdrs = glob([
        "tools/clang/include/**/*.inc",
    ]),
    defines = llvm_defines,
    includes = ["tools/clang/include"],
    deps = [
        ":config",
    ],
)

table_gen_library(
    name = "DiagnosticAnalysisKinds",
    extra_args = [
        "-clang-component=Analysis",
        "-I",
        "external/directx_shader_compiler/tools/clang/include/clang/Basic",
    ],
    tbl_outs = [(
        "-gen-clang-diags-defs",
        "tools/clang/include/clang/Basic/DiagnosticAnalysisKinds.inc",
    )],
    tblgen = ":clang_tblgen",
    td_file = "tools/clang/include/clang/Basic/Diagnostic.td",
    td_srcs = glob([
        "tools/clang/include/clang/Basic/*.td",
    ]),
)

table_gen_library(
    name = "DiagnosticCommonKinds",
    extra_args = [
        "-clang-component=Common",
        "-I",
        "external/directx_shader_compiler/tools/clang/include/clang/Basic",
    ],
    tbl_outs = [(
        "-gen-clang-diags-defs",
        "tools/clang/include/clang/Basic/DiagnosticCommonKinds.inc",
    )],
    tblgen = ":clang_tblgen",
    td_file = "tools/clang/include/clang/Basic/Diagnostic.td",
    td_srcs = glob([
        "tools/clang/include/clang/Basic/*.td",
    ]),
)

table_gen_library(
    name = "DiagnosticASTKinds",
    extra_args = [
        "-clang-component=AST",
        "-I",
        "external/directx_shader_compiler/tools/clang/include/clang/Basic",
    ],
    tbl_outs = [(
        "-gen-clang-diags-defs",
        "tools/clang/include/clang/Basic/DiagnosticASTKinds.inc",
    )],
    tblgen = ":clang_tblgen",
    td_file = "tools/clang/include/clang/Basic/Diagnostic.td",
    td_srcs = glob([
        "tools/clang/include/clang/Basic/*.td",
    ]),
)

table_gen_library(
    name = "DiagnosticCommentKinds",
    extra_args = [
        "-clang-component=Comment",
        "-I",
        "external/directx_shader_compiler/tools/clang/include/clang/Basic",
    ],
    tbl_outs = [(
        "-gen-clang-diags-defs",
        "tools/clang/include/clang/Basic/DiagnosticCommentKinds.inc",
    )],
    tblgen = ":clang_tblgen",
    td_file = "tools/clang/include/clang/Basic/Diagnostic.td",
    td_srcs = glob([
        "tools/clang/include/clang/Basic/*.td",
    ]),
)

table_gen_library(
    name = "DiagnosticDriverKinds",
    extra_args = [
        "-clang-component=Driver",
        "-I",
        "external/directx_shader_compiler/tools/clang/include/clang/Basic",
    ],
    tbl_outs = [(
        "-gen-clang-diags-defs",
        "tools/clang/include/clang/Basic/DiagnosticDriverKinds.inc",
    )],
    tblgen = ":clang_tblgen",
    td_file = "tools/clang/include/clang/Basic/Diagnostic.td",
    td_srcs = glob([
        "tools/clang/include/clang/Basic/*.td",
    ]),
)

table_gen_library(
    name = "DiagnosticFrontendKinds",
    extra_args = [
        "-clang-component=Frontend",
        "-I",
        "external/directx_shader_compiler/tools/clang/include/clang/Basic",
    ],
    tbl_outs = [(
        "-gen-clang-diags-defs",
        "tools/clang/include/clang/Basic/DiagnosticFrontendKinds.inc",
    )],
    tblgen = ":clang_tblgen",
    td_file = "tools/clang/include/clang/Basic/Diagnostic.td",
    td_srcs = glob([
        "tools/clang/include/clang/Basic/*.td",
    ]),
)

table_gen_library(
    name = "DiagnosticLexKinds",
    extra_args = [
        "-clang-component=Lex",
        "-I",
        "external/directx_shader_compiler/tools/clang/include/clang/Basic",
    ],
    tbl_outs = [(
        "-gen-clang-diags-defs",
        "tools/clang/include/clang/Basic/DiagnosticLexKinds.inc",
    )],
    tblgen = ":clang_tblgen",
    td_file = "tools/clang/include/clang/Basic/Diagnostic.td",
    td_srcs = glob([
        "tools/clang/include/clang/Basic/*.td",
    ]),
)

table_gen_library(
    name = "DiagnosticParseKinds",
    extra_args = [
        "-clang-component=Parse",
        "-I",
        "external/directx_shader_compiler/tools/clang/include/clang/Basic",
    ],
    tbl_outs = [(
        "-gen-clang-diags-defs",
        "tools/clang/include/clang/Basic/DiagnosticParseKinds.inc",
    )],
    tblgen = ":clang_tblgen",
    td_file = "tools/clang/include/clang/Basic/Diagnostic.td",
    td_srcs = glob([
        "tools/clang/include/clang/Basic/*.td",
    ]),
)

table_gen_library(
    name = "DiagnosticSemaKinds",
    extra_args = [
        "-clang-component=Sema",
        "-I",
        "external/directx_shader_compiler/tools/clang/include/clang/Basic",
    ],
    tbl_outs = [(
        "-gen-clang-diags-defs",
        "tools/clang/include/clang/Basic/DiagnosticSemaKinds.inc",
    )],
    tblgen = ":clang_tblgen",
    td_file = "tools/clang/include/clang/Basic/Diagnostic.td",
    td_srcs = glob([
        "tools/clang/include/clang/Basic/*.td",
    ]),
)

table_gen_library(
    name = "DiagnosticSerializationKinds",
    extra_args = [
        "-clang-component=Serialization",
        "-I",
        "external/directx_shader_compiler/tools/clang/include/clang/Basic",
    ],
    tbl_outs = [(
        "-gen-clang-diags-defs",
        "tools/clang/include/clang/Basic/DiagnosticSerializationKinds.inc",
    )],
    tblgen = ":clang_tblgen",
    td_file = "tools/clang/include/clang/Basic/Diagnostic.td",
    td_srcs = glob([
        "tools/clang/include/clang/Basic/*.td",
    ]),
)

table_gen_library(
    name = "DiagnosticGroups",
    extra_args = [
        "-I",
        "external/directx_shader_compiler/tools/clang/include/clang/Basic",
    ],
    tbl_outs = [(
        "-gen-clang-diag-groups",
        "tools/clang/include/clang/Basic/DiagnosticGroups.inc",
    )],
    tblgen = ":clang_tblgen",
    td_file = "tools/clang/include/clang/Basic/Diagnostic.td",
    td_srcs = glob([
        "tools/clang/include/clang/Basic/*.td",
    ]),
)

table_gen_library(
    name = "AttrList",
    tbl_outs = [(
        "-gen-clang-attr-list",
        "tools/clang/include/clang/Basic/AttrList.inc",
    )],
    tblgen = ":clang_tblgen",
    td_file = "tools/clang/include/clang/Basic/Attr.td",
    td_srcs = glob([
        "tools/clang/include/clang/Basic/*.td",
    ]),
)

table_gen_library(
    name = "AttrHasAttributeImpl",
    tbl_outs = [(
        "-gen-clang-attr-has-attribute-impl",
        "tools/clang/include/clang/Basic/AttrHasAttributeImpl.inc",
    )],
    tblgen = ":clang_tblgen",
    td_file = "tools/clang/include/clang/Basic/Attr.td",
    td_srcs = glob([
        "tools/clang/include/clang/Basic/*.td",
    ]),
)

table_gen_library(
    name = "Attrs",
    tbl_outs = [(
        "-gen-clang-attr-classes",
        "tools/clang/include/clang/AST/Attrs.inc",
    )],
    tblgen = ":clang_tblgen",
    td_file = "tools/clang/include/clang/Basic/Attr.td",
    td_srcs = glob([
        "tools/clang/include/clang/Basic/*.td",
    ]),
)

table_gen_library(
    name = "AttrImpl",
    tbl_outs = [(
        "-gen-clang-attr-impl",
        "tools/clang/include/clang/AST/AttrImpl.inc",
    )],
    tblgen = ":clang_tblgen",
    td_file = "tools/clang/include/clang/Basic/Attr.td",
    td_srcs = glob([
        "tools/clang/include/clang/Basic/*.td",
    ]),
)

table_gen_library(
    name = "AttrDump",
    tbl_outs = [(
        "-gen-clang-attr-dump",
        "tools/clang/include/clang/AST/AttrDump.inc",
    )],
    tblgen = ":clang_tblgen",
    td_file = "tools/clang/include/clang/Basic/Attr.td",
    td_srcs = glob([
        "tools/clang/include/clang/Basic/*.td",
    ]),
)

table_gen_library(
    name = "AttrVisitor",
    tbl_outs = [(
        "-gen-clang-attr-ast-visitor",
        "tools/clang/include/clang/AST/AttrVisitor.inc",
    )],
    tblgen = ":clang_tblgen",
    td_file = "tools/clang/include/clang/Basic/Attr.td",
    td_srcs = glob([
        "tools/clang/include/clang/Basic/*.td",
    ]),
)

table_gen_library(
    name = "AttrPCHRead",
    tbl_outs = [(
        "-gen-clang-attr-pch-read",
        "tools/clang/include/clang/Basic/AttrPCHRead.inc",
    )],
    tblgen = ":clang_tblgen",
    td_file = "tools/clang/include/clang/Basic/Attr.td",
    td_srcs = glob([
        "tools/clang/include/clang/Basic/*.td",
    ]),
)

table_gen_library(
    name = "AttrPCHWrite",
    tbl_outs = [(
        "-gen-clang-attr-pch-write",
        "tools/clang/include/clang/Basic/AttrPCHWrite.inc",
    )],
    tblgen = ":clang_tblgen",
    td_file = "tools/clang/include/clang/Basic/Attr.td",
    td_srcs = glob([
        "tools/clang/include/clang/Basic/*.td",
    ]),
)

table_gen_library(
    name = "AttrTemplateInstantiate",
    tbl_outs = [(
        "-gen-clang-attr-template-instantiate",
        "tools/clang/include/clang/Sema/AttrTemplateInstantiate.inc",
    )],
    tblgen = ":clang_tblgen",
    td_file = "tools/clang/include/clang/Basic/Attr.td",
    td_srcs = glob([
        "tools/clang/include/clang/Basic/*.td",
    ]),
)

table_gen_library(
    name = "AttrParsedAttrList",
    tbl_outs = [(
        "-gen-clang-attr-parsed-attr-list",
        "tools/clang/include/clang/Sema/AttrParsedAttrList.inc",
    )],
    tblgen = ":clang_tblgen",
    td_file = "tools/clang/include/clang/Basic/Attr.td",
    td_srcs = glob([
        "tools/clang/include/clang/Basic/*.td",
    ]),
)

table_gen_library(
    name = "AttrParsedAttrKinds",
    tbl_outs = [(
        "-gen-clang-attr-parsed-attr-kinds",
        "tools/clang/include/clang/Sema/AttrParsedAttrKinds.inc",
    )],
    tblgen = ":clang_tblgen",
    td_file = "tools/clang/include/clang/Basic/Attr.td",
    td_srcs = glob([
        "tools/clang/include/clang/Basic/*.td",
    ]),
)

table_gen_library(
    name = "AttrSpellingListIndex",
    tbl_outs = [(
        "-gen-clang-attr-spelling-index",
        "tools/clang/include/clang/Sema/AttrSpellingListIndex.inc",
    )],
    tblgen = ":clang_tblgen",
    td_file = "tools/clang/include/clang/Basic/Attr.td",
    td_srcs = glob([
        "tools/clang/include/clang/Basic/*.td",
    ]),
)

table_gen_library(
    name = "AttrParsedAttrImpl",
    tbl_outs = [(
        "-gen-clang-attr-parsed-attr-impl",
        "tools/clang/include/clang/Sema/AttrParsedAttrImpl.inc",
    )],
    tblgen = ":clang_tblgen",
    td_file = "tools/clang/include/clang/Basic/Attr.td",
    td_srcs = glob([
        "tools/clang/include/clang/Basic/*.td",
    ]),
)

table_gen_library(
    name = "StmtNodes",
    tbl_outs = [(
        "-gen-clang-stmt-nodes",
        "tools/clang/include/clang/AST/StmtNodes.inc",
    )],
    tblgen = ":clang_tblgen",
    td_file = "tools/clang/include/clang/Basic/StmtNodes.td",
    td_srcs = glob([
        "tools/clang/include/clang/Basic/*.td",
    ]),
)

table_gen_library(
    name = "DeclNodes",
    tbl_outs = [(
        "-gen-clang-decl-nodes",
        "tools/clang/include/clang/AST/DeclNodes.inc",
    )],
    tblgen = ":clang_tblgen",
    td_file = "tools/clang/include/clang/Basic/DeclNodes.td",
    td_srcs = glob([
        "tools/clang/include/clang/Basic/*.td",
    ]),
)

table_gen_library(
    name = "CommentNodes",
    tbl_outs = [(
        "-gen-clang-comment-nodes",
        "tools/clang/include/clang/AST/CommentNodes.inc",
    )],
    tblgen = ":clang_tblgen",
    td_file = "tools/clang/include/clang/Basic/CommentNodes.td",
    td_srcs = glob([
        "tools/clang/include/clang/Basic/*.td",
    ]),
)

table_gen_library(
    name = "CommentHTMLTags",
    tbl_outs = [(
        "-gen-clang-comment-html-tags",
        "tools/clang/include/clang/AST/CommentHTMLTags.inc",
    )],
    tblgen = ":clang_tblgen",
    td_file = "tools/clang/include/clang/AST/CommentHTMLTags.td",
    td_srcs = glob([
        "tools/clang/include/clang/Basic/*.td",
    ]),
)

table_gen_library(
    name = "CommentHTMLTagsProperties",
    tbl_outs = [(
        "-gen-clang-comment-html-tags-properties",
        "tools/clang/include/clang/AST/CommentHTMLTagsProperties.inc",
    )],
    tblgen = ":clang_tblgen",
    td_file = "tools/clang/include/clang/AST/CommentHTMLTags.td",
    td_srcs = glob([
        "tools/clang/include/clang/Basic/*.td",
    ]),
)

table_gen_library(
    name = "CommentHTMLNamedCharacterReferences",
    tbl_outs = [(
        "-gen-clang-comment-html-named-character-references",
        "tools/clang/include/clang/AST/CommentHTMLNamedCharacterReferences.inc",
    )],
    tblgen = ":clang_tblgen",
    td_file = "tools/clang/include/clang/AST/CommentHTMLNamedCharacterReferences.td",
    td_srcs = glob([
        "tools/clang/include/clang/Basic/*.td",
    ]),
)

table_gen_library(
    name = "CommentCommandInfo",
    tbl_outs = [(
        "-gen-clang-comment-command-info",
        "tools/clang/include/clang/AST/CommentCommandInfo.inc",
    )],
    tblgen = ":clang_tblgen",
    td_file = "tools/clang/include/clang/AST/CommentCommands.td",
    td_srcs = glob([
        "tools/clang/include/clang/Basic/*.td",
    ]),
)

table_gen_library(
    name = "CommentCommandList",
    tbl_outs = [(
        "-gen-clang-comment-command-list",
        "tools/clang/include/clang/AST/CommentCommandList.inc",
    )],
    tblgen = ":clang_tblgen",
    td_file = "tools/clang/include/clang/AST/CommentCommands.td",
    td_srcs = glob([
        "tools/clang/include/clang/Basic/*.td",
    ]),
)

table_gen_library(
    name = "AttrParserStringSwitches",
    tbl_outs = [(
        "-gen-clang-attr-parser-string-switches",
        "tools/clang/include/clang/Parse/AttrParserStringSwitches.inc",
    )],
    tblgen = ":clang_tblgen",
    td_file = "tools/clang/include/clang/Basic/Attr.td",
    td_srcs = glob([
        "tools/clang/include/clang/Basic/*.td",
    ]),
)

table_gen_library(
    name = "intrinsic_gen",
    tbl_outs = [(
        "-gen-intrinsic",
        "include/llvm/IR/Intrinsics.gen",
    )],
    tblgen = ":llvm_tblgen",
    td_file = "include/llvm/IR/Intrinsics.td",
    td_srcs = glob([
        "include/llvm/CodeGen/*.td",
        "include/llvm/IR/Intrinsics*.td",
    ]),
)

table_gen_library(
    name = "TablegenHLSLOptions",
    tbl_outs = [(
        "-gen-opt-parser-defs",
        "include/dxc/Support/HLSLOptions.inc",
    )],
    tblgen = ":llvm_tblgen",
    td_file = "include/dxc/Support/HLSLOptions.td",
    td_srcs = [
        "include/llvm/Option/OptParser.td",
    ],
)

table_gen_library(
    name = "ClangDriverOptions",
    extra_args = [
        "-I",
        "external/directx_shader_compiler/tools/clang/include/clang/Driver",
    ],
    tbl_outs = [(
        "-gen-opt-parser-defs",
        "tools/clang/include/clang/Driver/Options.inc",
    )],
    tblgen = ":llvm_tblgen",
    td_file = "tools/clang/include/clang/Driver/Options.td",
    td_srcs = [
        "include/llvm/Option/OptParser.td",
    ],
)

cc_binary(
    name = "clang_tblgen",
    srcs = [
        "tools/clang/utils/TableGen/ClangASTNodesEmitter.cpp",
        "tools/clang/utils/TableGen/ClangAttrEmitter.cpp",
        "tools/clang/utils/TableGen/ClangCommentCommandInfoEmitter.cpp",
        "tools/clang/utils/TableGen/ClangCommentHTMLNamedCharacterReferenceEmitter.cpp",
        "tools/clang/utils/TableGen/ClangCommentHTMLTagsEmitter.cpp",
        "tools/clang/utils/TableGen/ClangDiagnosticsEmitter.cpp",
        "tools/clang/utils/TableGen/ClangSACheckersEmitter.cpp",
        "tools/clang/utils/TableGen/NeonEmitter.cpp",
        "tools/clang/utils/TableGen/TableGen.cpp",
    ],
    copts = [
        "-Wno-macro-redefined",
    ] + llvm_copts,
    linkopts = llvm_linkopts,
    stamp = 0,
    deps = [
        ":MSSupport",
        ":Support",
        ":TableGen",
        ":config",
    ],
)

cc_binary(
    name = "llvm_tblgen",
    srcs = [
        "utils/TableGen/TableGen.cpp",
        "utils/TableGen/AsmMatcherEmitter.cpp",
        "utils/TableGen/AsmWriterEmitter.cpp",
        "utils/TableGen/AsmWriterInst.cpp",
        "utils/TableGen/CTagsEmitter.cpp",
        "utils/TableGen/CallingConvEmitter.cpp",
        "utils/TableGen/CodeEmitterGen.cpp",
        "utils/TableGen/CodeGenDAGPatterns.cpp",
        "utils/TableGen/CodeGenInstruction.cpp",
        "utils/TableGen/CodeGenMapTable.cpp",
        "utils/TableGen/CodeGenRegisters.cpp",
        "utils/TableGen/CodeGenSchedule.cpp",
        "utils/TableGen/CodeGenTarget.cpp",
        "utils/TableGen/DAGISelEmitter.cpp",
        "utils/TableGen/DAGISelMatcher.cpp",
        "utils/TableGen/DAGISelMatcherEmitter.cpp",
        "utils/TableGen/DAGISelMatcherGen.cpp",
        "utils/TableGen/DAGISelMatcherOpt.cpp",
        "utils/TableGen/DFAPacketizerEmitter.cpp",
        "utils/TableGen/DisassemblerEmitter.cpp",
        "utils/TableGen/FastISelEmitter.cpp",
        "utils/TableGen/FixedLenDecoderEmitter.cpp",
        "utils/TableGen/InstrInfoEmitter.cpp",
        "utils/TableGen/IntrinsicEmitter.cpp",
        "utils/TableGen/OptParserEmitter.cpp",
        "utils/TableGen/PseudoLoweringEmitter.cpp",
        "utils/TableGen/RegisterInfoEmitter.cpp",
        "utils/TableGen/SubtargetEmitter.cpp",
    ] + glob([
        "utils/TableGen/*.h",
    ]),
    copts = [
        "-Wno-macro-redefined",
    ] + llvm_copts,
    linkopts = llvm_linkopts,
    stamp = 0,
    deps = [
        ":MSSupport",
        ":Support",
        ":TableGen",
        ":config",
    ],
)

cc_binary(
    name = "FileCheck",
    testonly = 1,
    srcs = glob([
        "utils/FileCheck/*.cpp",
        "utils/FileCheck/*.h",
    ]),
    copts = llvm_copts,
    linkopts = llvm_linkopts,
    stamp = 0,
    deps = [":Support"],
)

py_binary(
    name = "lit",
    srcs = ["utils/lit/lit.py"] + glob(["utils/lit/lit/**/*.py"]),
)

cc_binary(
    name = "count",
    srcs = ["utils/count/count.c"],
)

cc_binary(
    name = "not",
    srcs = ["utils/not/not.cpp"],
    copts = llvm_copts,
    linkopts = llvm_linkopts,
    deps = [
        ":Support",
    ],
)

win_etw_library(
    name = "dxcetw",
    manifest = "include/dxc/Tracing/dxcetw.man",
    prefix = "DxcEtw_",
)

win_res_library(
    name = "dxcompiler_res",
    rc_file = "tools/clang/tools/dxcompiler/DXCompiler.rc",
    deps = [
        ":dxcetw",
    ],
)

cc_binary(
    name = "dxcompiler",
    srcs = select({
        "@toolchains//conditions:windows": [
            "tools/clang/tools/dxcompiler/dxcapi.cpp",
            "tools/clang/tools/dxcompiler/dxcassembler.cpp",
            "tools/clang/tools/dxcompiler/dxclibrary.cpp",
            "tools/clang/tools/dxcompiler/dxcompilerobj.cpp",
            "tools/clang/tools/dxcompiler/dxcvalidator.cpp",
            "tools/clang/tools/dxcompiler/DXCompiler.cpp",
            "tools/clang/tools/dxcompiler/dxcfilesystem.cpp",
            "tools/clang/tools/dxcompiler/dxillib.cpp",
            "tools/clang/tools/dxcompiler/dxcontainerbuilder.cpp",
            "tools/clang/tools/dxcompiler/dxcutil.cpp",
            "tools/clang/tools/dxcompiler/dxcdisassembler.cpp",
            "tools/clang/tools/dxcompiler/dxclinker.cpp",
        ],
        "@toolchains//conditions:linux": [
            "tools/clang/tools/dxcompiler/dxcapi.cpp",
            "tools/clang/tools/dxcompiler/dxcassembler.cpp",
            "tools/clang/tools/dxcompiler/dxclibrary.cpp",
            "tools/clang/tools/dxcompiler/dxcompilerobj.cpp",
            "tools/clang/tools/dxcompiler/DXCompiler.cpp",
            "tools/clang/tools/dxcompiler/dxcfilesystem.cpp",
            "tools/clang/tools/dxcompiler/dxcontainerbuilder.cpp",
            "tools/clang/tools/dxcompiler/dxcutil.cpp",
            "tools/clang/tools/dxcompiler/dxcdisassembler.cpp",
            "tools/clang/tools/dxcompiler/dxillib.cpp",
            "tools/clang/tools/dxcompiler/dxcvalidator.cpp",
        ],
    }) + glob([
        "tools/clang/tools/dxcompiler/*.inc",
        "tools/clang/tools/dxcompiler/*.h",
    ]),
    additional_linker_inputs = select({
        "@toolchains//conditions:windows": [
            "tools/clang/tools/dxcompiler/DXCompiler.def",
        ],
        "@toolchains//conditions:linux": [
            "tools/clang/tools/dxcompiler/DXCompiler.lds",
        ],
    }),
    copts = [
        "-Wno-switch",
        "-Wno-constant-logical-operand",
        "-Wno-inconsistent-missing-override",
        "-Wno-unused-function",
    ] + llvm_copts,
    linkopts = llvm_linkopts + select({
        "@toolchains//conditions:windows": [
            "/DEF:$(location tools/clang/tools/dxcompiler/DXCompiler.def)",
        ],
        "@toolchains//conditions:linux": [
            "-Wl,--version-script=$(location tools/clang/tools/dxcompiler/DXCompiler.lds)",
            "-Wl,--no-undefined",
            "-Wl,--no-allow-shlib-undefined",
        ],
    }),
    linkshared = True,
    local_defines = [
        "_CINDEX_LIB_",
    ],
    deps = [
        ":Analysis",
        ":AsmParser",
        ":BitReader",
        ":BitWriter",
        ":Core",
        ":DXIL",
        ":DxcSupport",
        ":DxilContainer",
        ":DxilPIXPasses",
        ":DxilRootSignature",
        ":HLSL",
        ":IPA",
        ":IPO",
        ":IRReader",
        ":InstCombine",
        ":LTO",
        ":Linker",
        ":MSSupport",
        ":Option",
        ":ProfileData",
        ":Scalar",
        ":Support",
        ":Target",
        ":TransformUtils",
        ":Vectorize",
        ":PassPrinters",
        ":clang",
        ":clangAST",
        ":clangAnalysis",
        ":clangBasic",
        ":clangCodeGen",
        ":clangDriver",
        ":clangEdit",
        ":clangFrontend",
        ":clangIndex",
        ":clangLex",
        ":clangRewrite",
        ":clangRewriteFrontend",
        ":clangSema",
        ":clangTooling",
        ":clangSPIRV",
    ] + select({
        "@toolchains//conditions:windows": [
            ":DxilDia",
            ":dxcetw",
            ":dxcompiler_res",
        ],
        "@toolchains//conditions:linux": [],
    }),
)

cc_library(
    name = "clang",
    srcs = [
        "tools/clang/tools/libclang/CIndex.cpp",
        "tools/clang/tools/libclang/CIndexCXX.cpp",
        "tools/clang/tools/libclang/CIndexCodeCompletion.cpp",
        "tools/clang/tools/libclang/CIndexDiagnostic.cpp",
        "tools/clang/tools/libclang/CIndexHigh.cpp",
        "tools/clang/tools/libclang/CIndexInclusionStack.cpp",
        "tools/clang/tools/libclang/CIndexUSRs.cpp",
        "tools/clang/tools/libclang/CIndexer.cpp",
        "tools/clang/tools/libclang/CXComment.cpp",
        "tools/clang/tools/libclang/CXCursor.cpp",
        "tools/clang/tools/libclang/CXCompilationDatabase.cpp",
        "tools/clang/tools/libclang/CXLoadedDiagnostic.cpp",
        "tools/clang/tools/libclang/CXSourceLocation.cpp",
        "tools/clang/tools/libclang/CXStoredDiagnostic.cpp",
        "tools/clang/tools/libclang/CXString.cpp",
        "tools/clang/tools/libclang/CXType.cpp",
        "tools/clang/tools/libclang/IndexBody.cpp",
        "tools/clang/tools/libclang/IndexDecl.cpp",
        "tools/clang/tools/libclang/IndexTypeSourceInfo.cpp",
        "tools/clang/tools/libclang/Indexing.cpp",
        "tools/clang/tools/libclang/IndexingContext.cpp",
        "tools/clang/tools/libclang/dxcisenseimpl.cpp",  # HLSL Change
        "tools/clang/tools/libclang/dxcrewriteunused.cpp",  # HLSL Change
    ] + glob([
        "tools/clang/tools/libclang/*.inc",
        "tools/clang/tools/libclang/*.h",
    ]),
    hdrs = glob([
        "tools/clang/tools/libclang/*.h",
        "tools/clang/tools/libclang/*.def",
        "tools/clang/tools/libclang/*.inc",
    ]),
    copts = [
        "-Wno-switch",
        "-Wno-constant-logical-operand",
        "-Wno-unused-function",
    ] + llvm_copts,
    local_defines = [
        "CINDEX_LINKAGE=",
        "LIBCLANG_CC=",
    ],
    deps = [
        ":Core",
        ":MSSupport",
        ":Support",
        ":clangAST",
        ":clangBasic",
        ":clangFrontend",
        ":clangIndex",
        ":clangLex",
        ":clangSema",
        ":clangTooling",
    ],
)

cc_library(
    name = "clangBasic",
    srcs = [
        "tools/clang/lib/Basic/Attributes.cpp",
        "tools/clang/lib/Basic/Builtins.cpp",
        "tools/clang/lib/Basic/CharInfo.cpp",
        "tools/clang/lib/Basic/Diagnostic.cpp",
        "tools/clang/lib/Basic/DiagnosticIDs.cpp",
        "tools/clang/lib/Basic/DiagnosticOptions.cpp",
        "tools/clang/lib/Basic/FileManager.cpp",
        "tools/clang/lib/Basic/FileSystemStatCache.cpp",
        "tools/clang/lib/Basic/IdentifierTable.cpp",
        "tools/clang/lib/Basic/LangOptions.cpp",
        "tools/clang/lib/Basic/Module.cpp",
        "tools/clang/lib/Basic/ObjCRuntime.cpp",
        "tools/clang/lib/Basic/OpenMPKinds.cpp",
        "tools/clang/lib/Basic/OperatorPrecedence.cpp",
        "tools/clang/lib/Basic/SanitizerBlacklist.cpp",
        "tools/clang/lib/Basic/Sanitizers.cpp",
        "tools/clang/lib/Basic/SourceLocation.cpp",
        "tools/clang/lib/Basic/SourceManager.cpp",
        "tools/clang/lib/Basic/TargetInfo.cpp",
        "tools/clang/lib/Basic/Targets.cpp",
        "tools/clang/lib/Basic/TokenKinds.cpp",
        "tools/clang/lib/Basic/Version.cpp",
        "tools/clang/lib/Basic/VersionTuple.cpp",
        "tools/clang/lib/Basic/VirtualFileSystem.cpp",
        "tools/clang/lib/Basic/Warnings.cpp",
    ] + glob([
        "tools/clang/lib/Basic/*.inc",
        "tools/clang/lib/Basic/*.h",
    ]) + [
        "tools/clang/include/clang/Basic/AttrHasAttributeImpl.inc",
        "tools/clang/include/clang/Basic/Version.inc",
        "tools/clang/include/clang/Config/config.h",
    ],
    hdrs = glob([
        "tools/clang/lib/Basic/*.h",
        "tools/clang/lib/Basic/*.def",
        "tools/clang/lib/Basic/*.inc",
    ]),
    copts = [
        "-Wno-constant-logical-operand",
    ] + llvm_copts,
    deps = [
        ":AttrHasAttributeImpl",
        ":AttrList",
        ":Core",
        ":DiagnosticASTKinds",
        ":DiagnosticAnalysisKinds",
        ":DiagnosticCommentKinds",
        ":DiagnosticCommonKinds",
        ":DiagnosticDriverKinds",
        ":DiagnosticFrontendKinds",
        ":DiagnosticGroups",
        ":DiagnosticLexKinds",
        ":DiagnosticParseKinds",
        ":DiagnosticSemaKinds",
        ":DiagnosticSerializationKinds",
        ":Support",
        ":clang_config",
    ],
)

cc_library(
    name = "clangLex",
    srcs = [
        "tools/clang/lib/Lex/HeaderMap.cpp",
        "tools/clang/lib/Lex/HeaderSearch.cpp",
        "tools/clang/lib/Lex/HLSLMacroExpander.cpp",
        "tools/clang/lib/Lex/Lexer.cpp",
        "tools/clang/lib/Lex/LiteralSupport.cpp",
        "tools/clang/lib/Lex/MacroArgs.cpp",
        "tools/clang/lib/Lex/MacroInfo.cpp",
        "tools/clang/lib/Lex/ModuleMap.cpp",
        "tools/clang/lib/Lex/PPCaching.cpp",
        "tools/clang/lib/Lex/PPCallbacks.cpp",
        "tools/clang/lib/Lex/PPConditionalDirectiveRecord.cpp",
        "tools/clang/lib/Lex/PPDirectives.cpp",
        "tools/clang/lib/Lex/PPExpressions.cpp",
        "tools/clang/lib/Lex/PPLexerChange.cpp",
        "tools/clang/lib/Lex/PPMacroExpansion.cpp",
        "tools/clang/lib/Lex/PTHLexer.cpp",
        "tools/clang/lib/Lex/Pragma.cpp",
        "tools/clang/lib/Lex/PreprocessingRecord.cpp",
        "tools/clang/lib/Lex/Preprocessor.cpp",
        "tools/clang/lib/Lex/PreprocessorLexer.cpp",
        "tools/clang/lib/Lex/ScratchBuffer.cpp",
        "tools/clang/lib/Lex/TokenConcatenation.cpp",
        "tools/clang/lib/Lex/TokenLexer.cpp",
    ] + glob([
        "tools/clang/lib/Lex/*.inc",
        "tools/clang/lib/Lex/*.h",
    ]),
    hdrs = glob([
        "tools/clang/lib/Lex/*.h",
        "tools/clang/lib/Lex/*.def",
        "tools/clang/lib/Lex/*.inc",
    ]),
    copts = [
        "-Wno-constant-logical-operand",
    ] + llvm_copts,
    deps = [
        ":Support",
        ":clangBasic",
        ":clang_config",
    ],
)

cc_library(
    name = "clangAnalysis",
    srcs = [
        "tools/clang/lib/Analysis/AnalysisDeclContext.cpp",
        "tools/clang/lib/Analysis/BodyFarm.cpp",
        "tools/clang/lib/Analysis/CFG.cpp",
        "tools/clang/lib/Analysis/CFGReachabilityAnalysis.cpp",
        "tools/clang/lib/Analysis/CFGStmtMap.cpp",
        "tools/clang/lib/Analysis/CallGraph.cpp",
        "tools/clang/lib/Analysis/Consumed.cpp",
        "tools/clang/lib/Analysis/CodeInjector.cpp",
        "tools/clang/lib/Analysis/Dominators.cpp",
        "tools/clang/lib/Analysis/LiveVariables.cpp",
        "tools/clang/lib/Analysis/ObjCNoReturn.cpp",
        "tools/clang/lib/Analysis/PostOrderCFGView.cpp",
        "tools/clang/lib/Analysis/ProgramPoint.cpp",
        "tools/clang/lib/Analysis/PseudoConstantAnalysis.cpp",
        "tools/clang/lib/Analysis/ReachableCode.cpp",
        "tools/clang/lib/Analysis/ThreadSafety.cpp",
        "tools/clang/lib/Analysis/ThreadSafetyCommon.cpp",
        "tools/clang/lib/Analysis/ThreadSafetyLogical.cpp",
        "tools/clang/lib/Analysis/ThreadSafetyTIL.cpp",
        "tools/clang/lib/Analysis/UninitializedValues.cpp",
    ] + glob([
        "tools/clang/lib/Analysis/*.inc",
        "tools/clang/lib/Analysis/*.h",
    ]),
    hdrs = glob([
        "tools/clang/lib/Analysis/*.h",
        "tools/clang/lib/Analysis/*.def",
        "tools/clang/lib/Analysis/*.inc",
    ]),
    copts = [
        "-Wno-constant-logical-operand",
    ] + llvm_copts,
    deps = [
        ":Support",
        ":clangAST",
        ":clangBasic",
        ":clangLex",
        ":clang_config",
    ],
)

cc_library(
    name = "clangAST",
    srcs = [
        "tools/clang/lib/AST/APValue.cpp",
        "tools/clang/lib/AST/ASTConsumer.cpp",
        "tools/clang/lib/AST/ASTContext.cpp",
        "tools/clang/lib/AST/ASTContextHLSL.cpp",
        "tools/clang/lib/AST/ASTDiagnostic.cpp",
        "tools/clang/lib/AST/ASTDumper.cpp",
        "tools/clang/lib/AST/ASTImporter.cpp",
        "tools/clang/lib/AST/ASTTypeTraits.cpp",
        "tools/clang/lib/AST/AttrImpl.cpp",
        "tools/clang/lib/AST/CXXInheritance.cpp",
        "tools/clang/lib/AST/Comment.cpp",
        "tools/clang/lib/AST/CommentBriefParser.cpp",
        "tools/clang/lib/AST/CommentCommandTraits.cpp",
        "tools/clang/lib/AST/CommentLexer.cpp",
        "tools/clang/lib/AST/CommentParser.cpp",
        "tools/clang/lib/AST/CommentSema.cpp",
        "tools/clang/lib/AST/Decl.cpp",
        "tools/clang/lib/AST/DeclarationName.cpp",
        "tools/clang/lib/AST/DeclBase.cpp",
        "tools/clang/lib/AST/DeclCXX.cpp",
        "tools/clang/lib/AST/DeclFriend.cpp",
        "tools/clang/lib/AST/DeclGroup.cpp",
        "tools/clang/lib/AST/DeclObjC.cpp",
        "tools/clang/lib/AST/DeclOpenMP.cpp",
        "tools/clang/lib/AST/DeclPrinter.cpp",
        "tools/clang/lib/AST/DeclTemplate.cpp",
        "tools/clang/lib/AST/Expr.cpp",
        "tools/clang/lib/AST/ExprClassification.cpp",
        "tools/clang/lib/AST/ExprConstant.cpp",
        "tools/clang/lib/AST/ExprCXX.cpp",
        "tools/clang/lib/AST/ExternalASTSource.cpp",
        "tools/clang/lib/AST/HlslBuiltinTypeDeclBuilder.cpp",
        "tools/clang/lib/AST/HlslTypes.cpp",
        "tools/clang/lib/AST/InheritViz.cpp",
        "tools/clang/lib/AST/ItaniumCXXABI.cpp",
        "tools/clang/lib/AST/ItaniumMangle.cpp",
        "tools/clang/lib/AST/Mangle.cpp",
        "tools/clang/lib/AST/MicrosoftCXXABI.cpp",
        "tools/clang/lib/AST/MicrosoftMangle.cpp",
        "tools/clang/lib/AST/NestedNameSpecifier.cpp",
        "tools/clang/lib/AST/ParentMap.cpp",
        "tools/clang/lib/AST/RawCommentList.cpp",
        "tools/clang/lib/AST/RecordLayout.cpp",
        "tools/clang/lib/AST/RecordLayoutBuilder.cpp",
        "tools/clang/lib/AST/SelectorLocationsKind.cpp",
        "tools/clang/lib/AST/Stmt.cpp",
        "tools/clang/lib/AST/StmtIterator.cpp",
        "tools/clang/lib/AST/StmtPrinter.cpp",
        "tools/clang/lib/AST/StmtProfile.cpp",
        "tools/clang/lib/AST/StmtViz.cpp",
        "tools/clang/lib/AST/TemplateBase.cpp",
        "tools/clang/lib/AST/TemplateName.cpp",
        "tools/clang/lib/AST/Type.cpp",
        "tools/clang/lib/AST/TypeLoc.cpp",
        "tools/clang/lib/AST/TypePrinter.cpp",
        "tools/clang/lib/AST/VTableBuilder.cpp",
        "tools/clang/lib/AST/VTTBuilder.cpp",
    ] + glob([
        "tools/clang/lib/AST/*.inc",
        "tools/clang/lib/AST/*.h",
    ]),
    hdrs = glob([
        "tools/clang/lib/AST/*.h",
        "tools/clang/lib/AST/*.def",
        "tools/clang/lib/AST/*.inc",
    ]),
    copts = [
        "-Wno-switch",
        "-Wno-constant-logical-operand",
        "-Wno-binding-in-condition",
    ] + select({
        "@toolchains//conditions:ubuntu": [
            "-Wno-misleading-indentation",
        ],
        "//conditions:default": [],
    }) + llvm_copts,
    deps = [
        ":AttrDump",
        ":AttrImpl",
        ":AttrParsedAttrImpl",
        ":AttrParsedAttrKinds",
        ":AttrParsedAttrList",
        ":AttrSpellingListIndex",
        ":AttrTemplateInstantiate",
        ":AttrVisitor",
        ":Attrs",
        ":CommentCommandInfo",
        ":CommentCommandList",
        ":CommentHTMLNamedCharacterReferences",
        ":CommentHTMLTags",
        ":CommentHTMLTagsProperties",
        ":CommentNodes",
        ":DeclNodes",
        ":HLSL",
        ":StmtNodes",
        ":Support",
        ":clangBasic",
        ":clangLex",
        ":clangSema",
    ],
)

cc_library(
    name = "clangASTMatchers",
    srcs = [
        "tools/clang/lib/ASTMatchers/ASTMatchFinder.cpp",
        "tools/clang/lib/ASTMatchers/ASTMatchersInternal.cpp",
    ] + glob([
        "tools/clang/lib/ASTMatchers/*.inc",
        "tools/clang/lib/ASTMatchers/*.h",
    ]),
    hdrs = glob([
        "tools/clang/lib/ASTMatchers/*.h",
        "tools/clang/lib/ASTMatchers/*.def",
        "tools/clang/lib/ASTMatchers/*.inc",
    ]),
    copts = [
        "-Wno-constant-logical-operand",
    ] + llvm_copts,
    deps = [
        ":Support",
        ":clangAST",
        ":clangBasic",
    ],
)

cc_library(
    name = "clangCodeGen",
    srcs = [
        "tools/clang/lib/CodeGen/BackendUtil.cpp",
        "tools/clang/lib/CodeGen/CGAtomic.cpp",
        "tools/clang/lib/CodeGen/CGBlocks.cpp",
        "tools/clang/lib/CodeGen/CGBuiltin.cpp",
        "tools/clang/lib/CodeGen/CGCUDANV.cpp",
        "tools/clang/lib/CodeGen/CGCUDARuntime.cpp",
        "tools/clang/lib/CodeGen/CGCXX.cpp",
        "tools/clang/lib/CodeGen/CGCXXABI.cpp",
        "tools/clang/lib/CodeGen/CGCall.cpp",
        "tools/clang/lib/CodeGen/CGClass.cpp",
        "tools/clang/lib/CodeGen/CGCleanup.cpp",
        "tools/clang/lib/CodeGen/CGDebugInfo.cpp",
        "tools/clang/lib/CodeGen/CGDecl.cpp",
        "tools/clang/lib/CodeGen/CGDeclCXX.cpp",
        "tools/clang/lib/CodeGen/CGException.cpp",
        "tools/clang/lib/CodeGen/CGExpr.cpp",
        "tools/clang/lib/CodeGen/CGExprAgg.cpp",
        "tools/clang/lib/CodeGen/CGExprCXX.cpp",
        "tools/clang/lib/CodeGen/CGExprComplex.cpp",
        "tools/clang/lib/CodeGen/CGExprConstant.cpp",
        "tools/clang/lib/CodeGen/CGExprScalar.cpp",
        "tools/clang/lib/CodeGen/CGHLSLRuntime.cpp",
        "tools/clang/lib/CodeGen/CGHLSLMS.cpp",
        "tools/clang/lib/CodeGen/CGHLSLMSFinishCodeGen.cpp",
        "tools/clang/lib/CodeGen/CGHLSLRootSignature.cpp",
        "tools/clang/lib/CodeGen/CGLoopInfo.cpp",
        "tools/clang/lib/CodeGen/CGObjC.cpp",
        "tools/clang/lib/CodeGen/CGRecordLayoutBuilder.cpp",
        "tools/clang/lib/CodeGen/CGStmt.cpp",
        "tools/clang/lib/CodeGen/CGStmtOpenMP.cpp",
        "tools/clang/lib/CodeGen/CGVTT.cpp",
        "tools/clang/lib/CodeGen/CGVTables.cpp",
        "tools/clang/lib/CodeGen/CodeGenABITypes.cpp",
        "tools/clang/lib/CodeGen/CodeGenAction.cpp",
        "tools/clang/lib/CodeGen/CodeGenFunction.cpp",
        "tools/clang/lib/CodeGen/CodeGenModule.cpp",
        "tools/clang/lib/CodeGen/CodeGenPGO.cpp",
        "tools/clang/lib/CodeGen/CodeGenTBAA.cpp",
        "tools/clang/lib/CodeGen/CodeGenTypes.cpp",
        "tools/clang/lib/CodeGen/CoverageMappingGen.cpp",
        "tools/clang/lib/CodeGen/ItaniumCXXABI.cpp",
        "tools/clang/lib/CodeGen/MicrosoftCXXABI.cpp",
        "tools/clang/lib/CodeGen/ModuleBuilder.cpp",
        "tools/clang/lib/CodeGen/ObjectFilePCHContainerOperations.cpp",
        "tools/clang/lib/CodeGen/SanitizerMetadata.cpp",
        "tools/clang/lib/CodeGen/TargetInfo.cpp",
    ] + glob([
        "tools/clang/lib/CodeGen/*.inc",
        "tools/clang/lib/CodeGen/*.h",
    ]),
    hdrs = glob([
        "tools/clang/lib/CodeGen/*.h",
        "tools/clang/lib/CodeGen/*.def",
        "tools/clang/lib/CodeGen/*.inc",
    ]),
    copts = [
        "-Wno-braced-scalar-init",
        "-Wno-switch",
        "-Wno-string-plus-int",
        "-Wno-constant-logical-operand",
        "-Wno-unknown-pragmas",
        "-Wno-unused-lambda-capture",
        "-Wno-pessimizing-move",
        "-Wno-binding-in-condition",
    ] + select({
        "@toolchains//conditions:ubuntu": [
            "-Wno-misleading-indentation",
        ],
        "//conditions:default": [],
    }) + llvm_copts,
    deps = [
        ":Analysis",
        ":BitReader",
        ":BitWriter",
        ":Core",
        ":DXIL",
        ":DxilRootSignature",
        ":IPO",
        ":IRReader",
        ":InstCombine",
        ":Linker",
        ":ProfileData",
        ":Scalar",
        ":Support",
        ":Target",
        ":TransformUtils",
        ":clangAST",
        ":clangBasic",
        ":clangFrontend",
        ":clangLex",
    ],
)

cc_library(
    name = "clangFrontend",
    srcs = [
        "tools/clang/lib/Frontend/ASTConsumers.cpp",
        "tools/clang/lib/Frontend/ASTMerge.cpp",
        "tools/clang/lib/Frontend/ASTUnit.cpp",
        "tools/clang/lib/Frontend/CacheTokens.cpp",
        "tools/clang/lib/Frontend/ChainedDiagnosticConsumer.cpp",
        "tools/clang/lib/Frontend/CodeGenOptions.cpp",
        "tools/clang/lib/Frontend/CompilerInstance.cpp",
        "tools/clang/lib/Frontend/CompilerInvocation.cpp",
        "tools/clang/lib/Frontend/CreateInvocationFromCommandLine.cpp",
        "tools/clang/lib/Frontend/DependencyFile.cpp",
        "tools/clang/lib/Frontend/DependencyGraph.cpp",
        "tools/clang/lib/Frontend/DiagnosticRenderer.cpp",
        "tools/clang/lib/Frontend/FrontendAction.cpp",
        "tools/clang/lib/Frontend/FrontendActions.cpp",
        "tools/clang/lib/Frontend/FrontendOptions.cpp",
        "tools/clang/lib/Frontend/HeaderIncludeGen.cpp",
        "tools/clang/lib/Frontend/InitHeaderSearch.cpp",
        "tools/clang/lib/Frontend/InitPreprocessor.cpp",
        "tools/clang/lib/Frontend/LangStandards.cpp",
        "tools/clang/lib/Frontend/LayoutOverrideSource.cpp",
        "tools/clang/lib/Frontend/LogDiagnosticPrinter.cpp",
        "tools/clang/lib/Frontend/ModuleDependencyCollector.cpp",
        "tools/clang/lib/Frontend/MultiplexConsumer.cpp",
        "tools/clang/lib/Frontend/PCHContainerOperations.cpp",
        "tools/clang/lib/Frontend/PrintPreprocessedOutput.cpp",
        "tools/clang/lib/Frontend/SerializedDiagnosticPrinter.cpp",
        "tools/clang/lib/Frontend/SerializedDiagnosticReader.cpp",
        "tools/clang/lib/Frontend/TextDiagnostic.cpp",
        "tools/clang/lib/Frontend/TextDiagnosticBuffer.cpp",
        "tools/clang/lib/Frontend/TextDiagnosticPrinter.cpp",
        "tools/clang/lib/Frontend/VerifyDiagnosticConsumer.cpp",
    ] + glob([
        "tools/clang/lib/Frontend/*.inc",
        "tools/clang/lib/Frontend/*.h",
    ]),
    hdrs = glob([
        "tools/clang/lib/Frontend/*.h",
        "tools/clang/lib/Frontend/*.def",
        "tools/clang/lib/Frontend/*.inc",
    ]),
    copts = [
        "-Wno-constant-logical-operand",
    ] + llvm_copts,
    deps = [
        ":BitReader",
        ":ClangDriverOptions",
        ":Option",
        ":Support",
        ":TablegenHLSLOptions",
        ":clangAST",
        ":clangBasic",
        ":clangDriver",
        ":clangEdit",
        ":clangLex",
        ":clangParse",
        ":clangSema",
        ":hlsl_dxcversion",
    ],
)

cc_library(
    name = "clangFrontendTool",
    srcs = [
        "tools/clang/lib/FrontendTool/ExecuteCompilerInvocation.cpp",
    ] + glob([
        "tools/clang/lib/FrontendTool/*.inc",
        "tools/clang/lib/FrontendTool/*.h",
    ]),
    hdrs = glob([
        "tools/clang/lib/FrontendTool/*.h",
        "tools/clang/lib/FrontendTool/*.def",
        "tools/clang/lib/FrontendTool/*.inc",
    ]),
    copts = [
        "-Wno-constant-logical-operand",
    ] + llvm_copts,
    deps = [
        ":ClangDriverOptions",
        ":Option",
        ":Support",
        ":clangBasic",
        ":clangCodeGen",
        ":clangDriver",
        ":clangFrontend",
        ":clangRewriteFrontend",
    ],
)

cc_library(
    name = "clangRewrite",
    srcs = [
        "tools/clang/lib/Rewrite/DeltaTree.cpp",
        "tools/clang/lib/Rewrite/HTMLRewrite.cpp",
        "tools/clang/lib/Rewrite/RewriteRope.cpp",
        "tools/clang/lib/Rewrite/Rewriter.cpp",
        "tools/clang/lib/Rewrite/TokenRewriter.cpp",
    ] + glob([
        "tools/clang/lib/Rewrite/*.inc",
        "tools/clang/lib/Rewrite/*.h",
    ]),
    hdrs = glob([
        "tools/clang/lib/Rewrite/*.h",
        "tools/clang/lib/Rewrite/*.def",
        "tools/clang/lib/Rewrite/*.inc",
    ]),
    copts = [
        "-Wno-constant-logical-operand",
    ] + llvm_copts,
    deps = [
        ":Support",
        ":clangBasic",
        ":clangLex",
    ],
)

cc_library(
    name = "clangRewriteFrontend",
    srcs = [
        "tools/clang/lib/Frontend/Rewrite/FixItRewriter.cpp",
        "tools/clang/lib/Frontend/Rewrite/FrontendActions_rewrite.cpp",
        "tools/clang/lib/Frontend/Rewrite/HTMLPrint.cpp",
        "tools/clang/lib/Frontend/Rewrite/InclusionRewriter.cpp",
        "tools/clang/lib/Frontend/Rewrite/RewriteMacros.cpp",
        "tools/clang/lib/Frontend/Rewrite/RewriteObjC.cpp",
        "tools/clang/lib/Frontend/Rewrite/RewriteTest.cpp",
    ] + glob([
        "tools/clang/lib/Frontend/Rewrite/*.inc",
        "tools/clang/lib/Frontend/Rewrite/*.h",
    ]),
    hdrs = glob([
        "tools/clang/lib/Frontend/Rewrite/*.h",
        "tools/clang/lib/Frontend/Rewrite/*.def",
        "tools/clang/lib/Frontend/Rewrite/*.inc",
    ]),
    copts = [
        "-Wno-constant-logical-operand",
    ] + llvm_copts,
    deps = [
        ":Support",
        ":clangAST",
        ":clangDriver",
        ":clangEdit",
        ":clangFrontend",
        ":clangLex",
        ":clangRewrite",
    ],
)

cc_library(
    name = "clangDriver",
    srcs = [
        "tools/clang/lib/Driver/DriverOptions.cpp",
    ] + glob([
        "tools/clang/lib/Driver/*.inc",
        "tools/clang/lib/Driver/*.h",
    ]),
    hdrs = glob([
        "tools/clang/lib/Driver/*.h",
        "tools/clang/lib/Driver/*.def",
        "tools/clang/lib/Driver/*.inc",
    ]),
    copts = [
        "-Wno-constant-logical-operand",
    ] + llvm_copts,
    deps = [
        ":ClangDriverOptions",
        ":Option",
        ":Support",
        ":clangBasic",
    ],
)

cc_library(
    name = "clangEdit",
    srcs = [
        "tools/clang/lib/Edit/Commit.cpp",
        "tools/clang/lib/Edit/EditedSource.cpp",
    ] + glob([
        "tools/clang/lib/Edit/*.inc",
        "tools/clang/lib/Edit/*.h",
    ]),
    hdrs = glob([
        "tools/clang/lib/Edit/*.h",
        "tools/clang/lib/Edit/*.def",
        "tools/clang/lib/Edit/*.inc",
    ]),
    copts = [
        "-Wno-constant-logical-operand",
    ] + llvm_copts,
    deps = [
        ":Support",
        ":clangAST",
        ":clangBasic",
        ":clangLex",
    ],
)

cc_library(
    name = "clangParse",
    srcs = [
        "tools/clang/lib/Parse/ParseAST.cpp",
        "tools/clang/lib/Parse/ParseCXXInlineMethods.cpp",
        "tools/clang/lib/Parse/ParseDecl.cpp",
        "tools/clang/lib/Parse/ParseDeclCXX.cpp",
        "tools/clang/lib/Parse/ParseExpr.cpp",
        "tools/clang/lib/Parse/ParseExprCXX.cpp",
        "tools/clang/lib/Parse/ParseInit.cpp",
        "tools/clang/lib/Parse/ParseObjc.cpp",
        "tools/clang/lib/Parse/ParseOpenMP.cpp",
        "tools/clang/lib/Parse/ParsePragma.cpp",
        "tools/clang/lib/Parse/ParseStmt.cpp",
        "tools/clang/lib/Parse/ParseStmtAsm.cpp",
        "tools/clang/lib/Parse/ParseTemplate.cpp",
        "tools/clang/lib/Parse/ParseTentative.cpp",
        "tools/clang/lib/Parse/Parser.cpp",
        "tools/clang/lib/Parse/ParseHLSL.cpp",
        "tools/clang/lib/Parse/HLSLRootSignature.cpp",
    ] + glob([
        "tools/clang/lib/Parse/*.inc",
        "tools/clang/lib/Parse/*.h",
    ]),
    hdrs = glob([
        "tools/clang/lib/Parse/*.h",
        "tools/clang/lib/Parse/*.def",
        "tools/clang/lib/Parse/*.inc",
    ]),
    copts = [
        "-Wno-constant-logical-operand",
        "-Wno-unknown-pragmas",
        "-Wno-unused-variable",
    ] + llvm_copts,
    deps = [
        ":DeclNodes",
        ":StmtNodes",
        ":Support",
        ":CommentCommandList",
        ":Attrs",
        ":AttrParsedAttrList",
        ":AttrParserStringSwitches",
        #":clangAST",
        ":clangBasic",
        #":clangLex",
        #":clangSema",
    ],
)

cc_library(
    name = "clangSema",
    srcs = [
        "tools/clang/lib/Sema/AnalysisBasedWarnings.cpp",
        "tools/clang/lib/Sema/AttributeList.cpp",
        "tools/clang/lib/Sema/CodeCompleteConsumer.cpp",
        "tools/clang/lib/Sema/DeclSpec.cpp",
        "tools/clang/lib/Sema/DelayedDiagnostic.cpp",
        "tools/clang/lib/Sema/IdentifierResolver.cpp",
        "tools/clang/lib/Sema/JumpDiagnostics.cpp",
        "tools/clang/lib/Sema/MultiplexExternalSemaSource.cpp",
        "tools/clang/lib/Sema/Scope.cpp",
        "tools/clang/lib/Sema/ScopeInfo.cpp",
        "tools/clang/lib/Sema/Sema.cpp",
        "tools/clang/lib/Sema/SemaAccess.cpp",
        "tools/clang/lib/Sema/SemaAttr.cpp",
        "tools/clang/lib/Sema/SemaCXXScopeSpec.cpp",
        "tools/clang/lib/Sema/SemaCast.cpp",
        "tools/clang/lib/Sema/SemaChecking.cpp",
        "tools/clang/lib/Sema/SemaCodeComplete.cpp",
        "tools/clang/lib/Sema/SemaConsumer.cpp",
        "tools/clang/lib/Sema/SemaCUDA.cpp",
        "tools/clang/lib/Sema/SemaDecl.cpp",
        "tools/clang/lib/Sema/SemaDeclAttr.cpp",
        "tools/clang/lib/Sema/SemaDeclCXX.cpp",
        "tools/clang/lib/Sema/SemaDeclObjC.cpp",
        "tools/clang/lib/Sema/SemaExceptionSpec.cpp",
        "tools/clang/lib/Sema/SemaExpr.cpp",
        "tools/clang/lib/Sema/SemaExprCXX.cpp",
        "tools/clang/lib/Sema/SemaExprMember.cpp",
        "tools/clang/lib/Sema/SemaExprObjC.cpp",
        "tools/clang/lib/Sema/SemaFixItUtils.cpp",
        "tools/clang/lib/Sema/SemaHLSL.cpp",
        "tools/clang/lib/Sema/SemaInit.cpp",
        "tools/clang/lib/Sema/SemaLambda.cpp",
        "tools/clang/lib/Sema/SemaLookup.cpp",
        "tools/clang/lib/Sema/SemaObjCProperty.cpp",
        "tools/clang/lib/Sema/SemaOpenMP.cpp",
        "tools/clang/lib/Sema/SemaOverload.cpp",
        "tools/clang/lib/Sema/SemaPseudoObject.cpp",
        "tools/clang/lib/Sema/SemaStmt.cpp",
        "tools/clang/lib/Sema/SemaStmtAsm.cpp",
        "tools/clang/lib/Sema/SemaStmtAttr.cpp",
        "tools/clang/lib/Sema/SemaTemplate.cpp",
        "tools/clang/lib/Sema/SemaTemplateDeduction.cpp",
        "tools/clang/lib/Sema/SemaTemplateInstantiate.cpp",
        "tools/clang/lib/Sema/SemaTemplateInstantiateDecl.cpp",
        "tools/clang/lib/Sema/SemaTemplateVariadic.cpp",
        "tools/clang/lib/Sema/SemaType.cpp",
        "tools/clang/lib/Sema/TypeLocBuilder.cpp",
    ] + glob([
        "tools/clang/lib/Sema/*.inc",
        "tools/clang/lib/Sema/*.h",
    ]),
    hdrs = glob([
        "tools/clang/lib/Sema/*.h",
        "tools/clang/lib/Sema/*.def",
        "tools/clang/lib/Sema/*.inc",
    ]),
    copts = [
        "-Wno-constant-logical-operand",
        "-Wno-switch",
        "-Wno-enum-compare-switch",
    ] + llvm_copts,
    deps = [
        ":AttrDump",
        ":AttrImpl",
        ":AttrParsedAttrImpl",
        ":AttrParsedAttrKinds",
        ":AttrParsedAttrList",
        ":AttrSpellingListIndex",
        ":AttrTemplateInstantiate",
        ":AttrVisitor",
        ":Attrs",
        ":CommentCommandInfo",
        ":CommentCommandList",
        ":CommentHTMLNamedCharacterReferences",
        ":CommentHTMLTags",
        ":CommentHTMLTagsProperties",
        ":CommentNodes",
        ":DeclNodes",
        ":HLSL",
        ":StmtNodes",
        ":Support",
        #":clangAST",
        #":clangAnalysis",
        ":clangBasic",
        #":clangEdit",
        ":clangLex",
    ],
)

cc_library(
    name = "clangToolingCore",
    srcs = [
        "tools/clang/lib/Tooling/Core/Replacement.cpp",
    ] + glob([
        "tools/clang/lib/Tooling/Core/*.inc",
        "tools/clang/lib/Tooling/Core/*.h",
    ]),
    hdrs = glob([
        "tools/clang/lib/Tooling/Core/*.h",
        "tools/clang/lib/Tooling/Core/*.def",
        "tools/clang/lib/Tooling/Core/*.inc",
    ]),
    copts = [
        "-Wno-constant-logical-operand",
    ] + llvm_copts,
    deps = [
        ":Support",
        ":clangBasic",
        ":clangLex",
        ":clangRewrite",
    ],
)

cc_library(
    name = "clangTooling",
    srcs = [
        "tools/clang/lib/Tooling/ArgumentsAdjusters.cpp",
        "tools/clang/lib/Tooling/CommonOptionsParser.cpp",
        "tools/clang/lib/Tooling/CompilationDatabase.cpp",
        "tools/clang/lib/Tooling/FileMatchTrie.cpp",
        "tools/clang/lib/Tooling/JSONCompilationDatabase.cpp",
        "tools/clang/lib/Tooling/Refactoring.cpp",
        "tools/clang/lib/Tooling/RefactoringCallbacks.cpp",
        "tools/clang/lib/Tooling/Tooling.cpp",
    ] + glob([
        "tools/clang/lib/Tooling/*.inc",
        "tools/clang/lib/Tooling/*.h",
    ]),
    hdrs = glob([
        "tools/clang/lib/Tooling/*.h",
        "tools/clang/lib/Tooling/*.def",
        "tools/clang/lib/Tooling/*.inc",
    ]),
    copts = [
        "-Wno-constant-logical-operand",
    ] + llvm_copts,
    deps = [
        ":Support",
        ":clangAST",
        ":clangASTMatchers",
        ":clangBasic",
        ":clangDriver",
        ":clangFrontend",
        ":clangLex",
        ":clangRewrite",
        ":clangToolingCore",
    ],
)

cc_library(
    name = "clangIndex",
    srcs = [
        "tools/clang/lib/Index/CommentToXML.cpp",
        "tools/clang/lib/Index/USRGeneration.cpp",
    ] + glob([
        "tools/clang/lib/Index/*.inc",
        "tools/clang/lib/Index/*.h",
    ]),
    hdrs = glob([
        "tools/clang/lib/Index/*.h",
        "tools/clang/lib/Index/*.def",
        "tools/clang/lib/Index/*.inc",
    ]),
    copts = [
        "-Wno-constant-logical-operand",
    ] + llvm_copts,
    deps = [
        ":Support",
        ":clangAST",
        ":clangBasic",
        ":clangFormat",
        ":clangRewrite",
        ":clangToolingCore",
    ],
)

cc_library(
    name = "clangFormat",
    srcs = [
        "tools/clang/lib/Format/BreakableToken.cpp",
        "tools/clang/lib/Format/ContinuationIndenter.cpp",
        "tools/clang/lib/Format/Format.cpp",
        "tools/clang/lib/Format/FormatToken.cpp",
        "tools/clang/lib/Format/TokenAnnotator.cpp",
        "tools/clang/lib/Format/UnwrappedLineFormatter.cpp",
        "tools/clang/lib/Format/UnwrappedLineParser.cpp",
        "tools/clang/lib/Format/WhitespaceManager.cpp",
    ] + glob([
        "tools/clang/lib/Format/*.inc",
        "tools/clang/lib/Format/*.h",
    ]),
    hdrs = glob([
        "tools/clang/lib/Format/*.h",
        "tools/clang/lib/Format/*.def",
        "tools/clang/lib/Format/*.inc",
    ]),
    copts = [
        "-Wno-constant-logical-operand",
    ] + llvm_copts,
    deps = [
        ":Support",
        ":clangBasic",
        ":clangLex",
        ":clangToolingCore",
    ],
)

cc_library(
    name = "clangSPIRV",
    srcs = [
        "tools/clang/lib/SPIRV/AlignmentSizeCalculator.cpp",
        "tools/clang/lib/SPIRV/AstTypeProbe.cpp",
        "tools/clang/lib/SPIRV/BlockReadableOrder.cpp",
        "tools/clang/lib/SPIRV/CapabilityVisitor.cpp",
        "tools/clang/lib/SPIRV/DeclResultIdMapper.cpp",
        "tools/clang/lib/SPIRV/EmitSpirvAction.cpp",
        "tools/clang/lib/SPIRV/EmitVisitor.cpp",
        "tools/clang/lib/SPIRV/FeatureManager.cpp",
        "tools/clang/lib/SPIRV/GlPerVertex.cpp",
        "tools/clang/lib/SPIRV/InitListHandler.cpp",
        "tools/clang/lib/SPIRV/LiteralTypeVisitor.cpp",
        "tools/clang/lib/SPIRV/LowerTypeVisitor.cpp",
        "tools/clang/lib/SPIRV/PreciseVisitor.cpp",
        "tools/clang/lib/SPIRV/RawBufferMethods.cpp",
        "tools/clang/lib/SPIRV/RelaxedPrecisionVisitor.cpp",
        "tools/clang/lib/SPIRV/SpirvBasicBlock.cpp",
        "tools/clang/lib/SPIRV/SpirvBuilder.cpp",
        "tools/clang/lib/SPIRV/SpirvContext.cpp",
        "tools/clang/lib/SPIRV/SpirvEmitter.cpp",
        "tools/clang/lib/SPIRV/SpirvFunction.cpp",
        "tools/clang/lib/SPIRV/SpirvInstruction.cpp",
        "tools/clang/lib/SPIRV/SpirvModule.cpp",
        "tools/clang/lib/SPIRV/SpirvType.cpp",
        "tools/clang/lib/SPIRV/String.cpp",
    ] + glob([
        "tools/clang/lib/SPIRV/*.inc",
        "tools/clang/lib/SPIRV/*.h",
    ]),
    hdrs = glob([
        "tools/clang/lib/SPIRV/*.h",
        "tools/clang/lib/SPIRV/*.def",
        "tools/clang/lib/SPIRV/*.inc",
    ]),
    copts = [
        "-Wno-parentheses",
        "-Wno-constant-logical-operand",
        "-Wno-unused-variable",
    ] + llvm_copts,
    deps = [
        ":Support",
        ":clangAST",
        ":clangBasic",
        ":clangFrontend",
        ":clangLex",
        "@spirv_tools//:spirv_tools_opt",
    ],
)

cc_library(
    name = "Analysis",
    srcs = [
        "lib/Analysis/AliasAnalysis.cpp",
        "lib/Analysis/AliasAnalysisCounter.cpp",
        "lib/Analysis/AliasAnalysisEvaluator.cpp",
        "lib/Analysis/AliasDebugger.cpp",
        "lib/Analysis/AliasSetTracker.cpp",
        "lib/Analysis/Analysis.cpp",
        "lib/Analysis/AssumptionCache.cpp",
        "lib/Analysis/BasicAliasAnalysis.cpp",
        "lib/Analysis/BlockFrequencyInfo.cpp",
        "lib/Analysis/BlockFrequencyInfoImpl.cpp",
        "lib/Analysis/BranchProbabilityInfo.cpp",
        "lib/Analysis/CFG.cpp",
        "lib/Analysis/CFGPrinter.cpp",
        "lib/Analysis/CFLAliasAnalysis.cpp",
        "lib/Analysis/CGSCCPassManager.cpp",
        "lib/Analysis/CaptureTracking.cpp",
        "lib/Analysis/CostModel.cpp",
        "lib/Analysis/CodeMetrics.cpp",
        "lib/Analysis/ConstantFolding.cpp",
        "lib/Analysis/Delinearization.cpp",
        "lib/Analysis/DependenceAnalysis.cpp",
        "lib/Analysis/DivergenceAnalysis.cpp",
        "lib/Analysis/DomPrinter.cpp",
        "lib/Analysis/DominanceFrontier.cpp",
        "lib/Analysis/DxilConstantFolding.cpp",
        "lib/Analysis/DxilConstantFoldingExt.cpp",
        "lib/Analysis/DxilSimplify.cpp",
        "lib/Analysis/DxilValueCache.cpp",
        "lib/Analysis/IVUsers.cpp",
        "lib/Analysis/InstCount.cpp",
        "lib/Analysis/InstructionSimplify.cpp",
        "lib/Analysis/Interval.cpp",
        "lib/Analysis/IntervalPartition.cpp",
        "lib/Analysis/IteratedDominanceFrontier.cpp",
        "lib/Analysis/LazyCallGraph.cpp",
        "lib/Analysis/LazyValueInfo.cpp",
        "lib/Analysis/LibCallAliasAnalysis.cpp",
        "lib/Analysis/LibCallSemantics.cpp",
        "lib/Analysis/Lint.cpp",
        "lib/Analysis/Loads.cpp",
        "lib/Analysis/LoopAccessAnalysis.cpp",
        "lib/Analysis/LoopInfo.cpp",
        "lib/Analysis/LoopPass.cpp",
        "lib/Analysis/MemDepPrinter.cpp",
        "lib/Analysis/MemDerefPrinter.cpp",
        "lib/Analysis/MemoryBuiltins.cpp",
        "lib/Analysis/MemoryDependenceAnalysis.cpp",
        "lib/Analysis/MemoryLocation.cpp",
        "lib/Analysis/ModuleDebugInfoPrinter.cpp",
        "lib/Analysis/NoAliasAnalysis.cpp",
        "lib/Analysis/PHITransAddr.cpp",
        "lib/Analysis/PostDominators.cpp",
        "lib/Analysis/PtrUseVisitor.cpp",
        "lib/Analysis/ReducibilityAnalysis.cpp",
        "lib/Analysis/regioninfo.cpp",
        "lib/Analysis/RegionPass.cpp",
        "lib/Analysis/regionprinter.cpp",
        "lib/Analysis/ScalarEvolution.cpp",
        "lib/Analysis/ScalarEvolutionAliasAnalysis.cpp",
        "lib/Analysis/ScalarEvolutionExpander.cpp",
        "lib/Analysis/ScalarEvolutionNormalization.cpp",
        "lib/Analysis/SparsePropagation.cpp",
        "lib/Analysis/TargetLibraryInfo.cpp",
        "lib/Analysis/TargetTransformInfo.cpp",
        "lib/Analysis/Trace.cpp",
        "lib/Analysis/TypeBasedAliasAnalysis.cpp",
        "lib/Analysis/ScopedNoAliasAA.cpp",
        "lib/Analysis/ValueTracking.cpp",
        "lib/Analysis/VectorUtils.cpp",
    ] + glob([
        "lib/Analysis/*.inc",
        "include/llvm/Transforms/Utils/Local.h",
        "include/llvm/Transforms/Scalar.h",
        "lib/Analysis/*.h",
    ]),
    hdrs = glob([
        "include/llvm/Analysis/*.h",
        "include/llvm/Analysis/*.def",
        "include/llvm/Analysis/*.inc",
    ]),
    copts = [
        "-Wno-inconsistent-missing-override",
        "-Wno-unused-variable",
    ] + llvm_copts,
    deps = [
        ":Core",
        ":ProfileData",
        ":Support",
        ":config",
    ],
)

cc_library(
    name = "AsmParser",
    srcs = [
        "lib/AsmParser/LLLexer.cpp",
        "lib/AsmParser/LLParser.cpp",
        "lib/AsmParser/Parser.cpp",
    ] + glob([
        "lib/AsmParser/*.inc",
        "lib/AsmParser/*.h",
    ]),
    hdrs = glob([
        "lib/AsmParser/*.h",
        "lib/AsmParser/*.def",
        "lib/AsmParser/*.inc",
    ]),
    copts = llvm_copts,
    deps = [
        ":Core",
        ":Support",
        ":config",
    ],
)

cc_library(
    name = "BitReader",
    srcs = [
        "lib/Bitcode/Reader/BitcodeReader.cpp",
        "lib/Bitcode/Reader/BitstreamReader.cpp",
    ] + glob([
        "lib/Bitcode/Reader/*.inc",
        "lib/Bitcode/Reader/*.h",
    ]),
    hdrs = glob([
        "lib/Bitcode/Reader/*.h",
        "lib/Bitcode/Reader/*.def",
        "lib/Bitcode/Reader/*.inc",
        "include/llvm/Bitcode/BitstreamReader.h",
    ]),
    copts = llvm_copts,
    deps = [
        ":Core",
        ":Support",
        ":config",
    ],
)

cc_library(
    name = "BitWriter",
    srcs = [
        "lib/Bitcode/Writer/BitWriter.cpp",
        "lib/Bitcode/Writer/BitcodeWriter.cpp",
        "lib/Bitcode/Writer/BitcodeWriterPass.cpp",
        "lib/Bitcode/Writer/ValueEnumerator.cpp",
    ] + glob([
        "lib/Bitcode/Writer/*.inc",
        "lib/Bitcode/Writer/*.h",
    ]),
    hdrs = glob([
        "lib/Bitcode/Writer/*.h",
        "lib/Bitcode/Writer/*.def",
        "lib/Bitcode/Writer/*.inc",
        "include/llvm/Bitcode/BitcodeWriter.h",
        "include/llvm/Bitcode/BitcodeWriterPass.h",
        "include/llvm/Bitcode/BitstreamWriter.h",
    ]),
    copts = llvm_copts,
    deps = [
        ":Core",
        ":Support",
        ":config",
    ],
)

cc_library(
    name = "Core",
    srcs = [
        "lib/IR/AsmWriter.cpp",
        "lib/IR/Attributes.cpp",
        "lib/IR/AutoUpgrade.cpp",
        "lib/IR/BasicBlock.cpp",
        "lib/IR/Comdat.cpp",
        "lib/IR/ConstantFold.cpp",
        "lib/IR/ConstantRange.cpp",
        "lib/IR/Constants.cpp",
        "lib/IR/Core.cpp",
        "lib/IR/DIBuilder.cpp",
        "lib/IR/DataLayout.cpp",
        "lib/IR/DebugInfo.cpp",
        "lib/IR/DebugInfoMetadata.cpp",
        "lib/IR/DebugLoc.cpp",
        "lib/IR/DiagnosticInfo.cpp",
        "lib/IR/DiagnosticPrinter.cpp",
        "lib/IR/Dominators.cpp",
        "lib/IR/Function.cpp",
        "lib/IR/GCOV.cpp",
        "lib/IR/GVMaterializer.cpp",
        "lib/IR/Globals.cpp",
        "lib/IR/IRBuilder.cpp",
        "lib/IR/IRPrintingPasses.cpp",
        "lib/IR/InlineAsm.cpp",
        "lib/IR/Instruction.cpp",
        "lib/IR/Instructions.cpp",
        "lib/IR/IntrinsicInst.cpp",
        "lib/IR/LLVMContext.cpp",
        "lib/IR/LLVMContextImpl.cpp",
        "lib/IR/LegacyPassManager.cpp",
        "lib/IR/MDBuilder.cpp",
        "lib/IR/Mangler.cpp",
        "lib/IR/Metadata.cpp",
        "lib/IR/MetadataTracking.cpp",
        "lib/IR/Module.cpp",
        "lib/IR/Operator.cpp",
        "lib/IR/Pass.cpp",
        "lib/IR/PassManager.cpp",
        "lib/IR/PassRegistry.cpp",
        "lib/IR/Statepoint.cpp",
        "lib/IR/Type.cpp",
        "lib/IR/TypeFinder.cpp",
        "lib/IR/Use.cpp",
        "lib/IR/User.cpp",
        "lib/IR/Value.cpp",
        "lib/IR/ValueSymbolTable.cpp",
        "lib/IR/ValueTypes.cpp",
        "lib/IR/Verifier.cpp",
    ] + glob([
        "lib/IR/*.inc",
        "include/llvm/Analysis/*.h",
        "include/llvm/Bitcode/BitcodeReader.h",
        "include/llvm/Bitcode/BitCodes.h",
        "include/llvm/Bitcode/LLVMBitCodes.h",
        "include/llvm/CodeGen/MachineValueType.h",
        "include/llvm/CodeGen/ValueTypes.h",
        "lib/IR/*.h",
    ]),
    hdrs = glob([
        "lib/IR/*.h",
        "lib/IR/*.def",
        "lib/IR/*.inc",
        "include/llvm/*.h",
        "include/llvm/Analysis/*.def",
    ]),
    copts = llvm_copts,
    deps = [
        ":Support",
        ":config",
        ":intrinsic_gen",
    ],
)

cc_library(
    name = "DxcSupport",
    srcs = [
        "lib/DxcSupport/dxcapi.use.cpp",
        "lib/DxcSupport/dxcmem.cpp",
        "lib/DxcSupport/FileIOHelper.cpp",
        "lib/DxcSupport/Global.cpp",
        "lib/DxcSupport/HLSLOptions.cpp",
        "lib/DxcSupport/Unicode.cpp",
        "lib/DxcSupport/WinAdapter.cpp",
        "lib/DxcSupport/WinFunctions.cpp",
    ] + glob([
        "lib/DxcSupport/*.inc",
        "lib/DxcSupport/*.h",
    ]),
    hdrs = glob([
        "lib/DxcSupport/*.h",
        "lib/DxcSupport/*.def",
        "lib/DxcSupport/*.inc",
    ]),
    copts = [
        "-Wno-inconsistent-missing-override",
        "-Wno-unused-function",
    ] + llvm_copts,
    deps = [
        ":Support",
        ":TablegenHLSLOptions",
        ":config",
    ],
)

cc_library(
    name = "DXIL",
    srcs = [
        "lib/DXIL/DxilCBuffer.cpp",
        "lib/DXIL/DxilCompType.cpp",
        "lib/DXIL/DxilInterpolationMode.cpp",
        "lib/DXIL/DxilMetadataHelper.cpp",
        "lib/DXIL/DxilModule.cpp",
        "lib/DXIL/DxilOperations.cpp",
        "lib/DXIL/DxilResource.cpp",
        "lib/DXIL/DxilResourceBase.cpp",
        "lib/DXIL/DxilResourceProperties.cpp",
        "lib/DXIL/DxilSampler.cpp",
        "lib/DXIL/DxilSemantic.cpp",
        "lib/DXIL/DxilShaderFlags.cpp",
        "lib/DXIL/DxilShaderModel.cpp",
        "lib/DXIL/DxilSignature.cpp",
        "lib/DXIL/DxilSignatureElement.cpp",
        "lib/DXIL/DxilSubobject.cpp",
        "lib/DXIL/DxilTypeSystem.cpp",
        "lib/DXIL/DxilUtil.cpp",
        "lib/DXIL/DxilPDB.cpp",
    ] + glob([
        "lib/DXIL/*.inc",
        "lib/DXIL/*.h",
    ]),
    hdrs = glob([
        "lib/DXIL/*.h",
        "lib/DXIL/*.def",
        "lib/DXIL/*.inc",
    ]),
    copts = [
        "-Wno-switch",
    ] + llvm_copts,
    deps = [
        ":BitReader",
        ":Core",
        ":DxcSupport",
        ":IPA",
        ":Support",
        ":config",
    ],
)

cc_library(
    name = "DxilContainer",
    srcs = [
        "lib/DxilContainer/DxilContainer.cpp",
        "lib/DxilContainer/DxilContainerAssembler.cpp",
        "lib/DxilContainer/DxilContainerReader.cpp",
        "lib/DxilContainer/DxilRuntimeReflection.cpp",
    ] + glob([
        "lib/DxilContainer/*.inc",
        "lib/DxilContainer/*.h",
    ]),
    hdrs = glob([
        "lib/DxilContainer/*.h",
        "lib/DxilContainer/*.def",
        "lib/DxilContainer/*.inc",
    ]),
    copts = [
        "-Wno-switch",
        "-Wno-pragma-once-outside-header",
    ] + llvm_copts,
    deps = [
        ":BitReader",
        ":Core",
        ":DxcSupport",
        ":IPA",
        ":Support",
        ":config",
    ],
)

cc_library(
    name = "DxilDia",
    srcs = [
        "lib/DxilDia/DxcPixCompilationInfo.cpp",
        "lib/DxilDia/DxcPixDxilDebugInfo.cpp",
        "lib/DxilDia/DxcPixDxilStorage.cpp",
        "lib/DxilDia/DxcPixEntrypoints.cpp",
        "lib/DxilDia/DxcPixLiveVariables.cpp",
        "lib/DxilDia/DxcPixLiveVariables_FragmentIterator.cpp",
        "lib/DxilDia/DxcPixTypes.cpp",
        "lib/DxilDia/DxcPixVariables.cpp",
        "lib/DxilDia/DxilDia.cpp",
        "lib/DxilDia/DxilDiaDataSource.cpp",
        "lib/DxilDia/DxilDiaEnumTables.cpp",
        "lib/DxilDia/DxilDiaSession.cpp",
        "lib/DxilDia/DxilDiaSymbolManager.cpp",
        "lib/DxilDia/DxilDiaTable.cpp",
        "lib/DxilDia/DxilDiaTableFrameData.cpp",
        "lib/DxilDia/DxilDiaTableInjectedSources.cpp",
        "lib/DxilDia/DxilDiaTableInputAssemblyFile.cpp",
        "lib/DxilDia/DxilDiaTableLineNumbers.cpp",
        "lib/DxilDia/DxilDiaTableSections.cpp",
        "lib/DxilDia/DxilDiaTableSegmentMap.cpp",
        "lib/DxilDia/DxilDiaTableSourceFiles.cpp",
        "lib/DxilDia/DxilDiaTableSymbols.cpp",
    ] + glob([
        "lib/DxilDia/*.inc",
        "lib/DxilDia/*.h",
    ]),
    hdrs = glob([
        "lib/DxilDia/*.h",
        "lib/DxilDia/*.def",
        "lib/DxilDia/*.inc",
    ]),
    copts = llvm_copts + [
        "-Wno-inconsistent-missing-override",
        "-Wno-bool-conversion",
        "-Wno-delete-abstract-non-virtual-dtor",
        "-Wno-enum-compare",
        "-Wno-trigraphs",
        "-Wno-switch",
        "-Wno-microsoft-template",
    ],
    deps = [
        ":Core",
        ":DxcSupport",
        ":DxilPIXPasses",
        ":Support",
        ":config",
    ],
)

cc_library(
    name = "DxilPIXPasses",
    srcs = [
        "lib/DxilPIXPasses/DxilAddPixelHitInstrumentation.cpp",
        "lib/DxilPIXPasses/DxilAnnotateWithVirtualRegister.cpp",
        "lib/DxilPIXPasses/DxilDbgValueToDbgDeclare.cpp",
        "lib/DxilPIXPasses/DxilDebugInstrumentation.cpp",
        "lib/DxilPIXPasses/DxilForceEarlyZ.cpp",
        "lib/DxilPIXPasses/DxilOutputColorBecomesConstant.cpp",
        "lib/DxilPIXPasses/DxilPIXMeshShaderOutputInstrumentation.cpp",
        "lib/DxilPIXPasses/DxilRemoveDiscards.cpp",
        "lib/DxilPIXPasses/DxilReduceMSAAToSingleSample.cpp",
        "lib/DxilPIXPasses/DxilShaderAccessTracking.cpp",
        "lib/DxilPIXPasses/DxilPIXPasses.cpp",
        "lib/DxilPIXPasses/DxilPIXVirtualRegisters.cpp",
    ] + glob([
        "lib/DxilPIXPasses/*.inc",
        "lib/DxilPIXPasses/*.h",
    ]),
    hdrs = glob([
        "lib/DxilPIXPasses/*.h",
        "lib/DxilPIXPasses/*.def",
        "lib/DxilPIXPasses/*.inc",
    ]),
    copts = [
        "-Wno-switch",
    ] + llvm_copts,
    deps = [
        ":BitReader",
        ":Core",
        ":DxcSupport",
        ":IPA",
        ":Support",
        ":config",
    ],
)

cc_library(
    name = "DxilRootSignature",
    srcs = [
        "lib/DxilRootSignature/DxilRootSignature.cpp",
        "lib/DxilRootSignature/DxilRootSignatureConvert.cpp",
        "lib/DxilRootSignature/DxilRootSignatureSerializer.cpp",
        "lib/DxilRootSignature/DxilRootSignatureValidator.cpp",
    ] + glob([
        "lib/DxilRootSignature/*.inc",
        "lib/DxilRootSignature/*.h",
    ]),
    hdrs = glob([
        "lib/DxilRootSignature/*.h",
        "lib/DxilRootSignature/*.def",
        "lib/DxilRootSignature/*.inc",
    ]),
    copts = llvm_copts,
    deps = [
        ":BitReader",
        ":Core",
        ":DxcSupport",
        ":IPA",
        ":Support",
        ":config",
    ],
)

cc_library(
    name = "DxrFallback",
    srcs = [
        "lib/DxrFallback/DxrFallbackCompiler.cpp",
        "lib/DxrFallback/LiveValues.cpp",
        "lib/DxrFallback/LLVMUtils.cpp",
        "lib/DxrFallback/Reducibility.cpp",
        "lib/DxrFallback/StateFunctionTransform.cpp",
    ] + glob([
        "lib/DxrFallback/*.inc",
        "lib/DxrFallback/*.h",
    ]),
    hdrs = glob([
        "lib/DxrFallback/*.h",
        "lib/DxrFallback/*.def",
        "lib/DxrFallback/*.inc",
    ]),
    copts = llvm_copts,
    deps = [
        ":Core",
        ":Support",
        ":config",
    ],
)

cc_library(
    name = "HLSL",
    srcs = [
        "lib/HLSL/ComputeViewIdState.cpp",
        "lib/HLSL/ComputeViewIdStateBuilder.cpp",
        "lib/HLSL/ControlDependence.cpp",
        "lib/HLSL/DxilCondenseResources.cpp",
        "lib/HLSL/DxilContainerReflection.cpp",
        "lib/HLSL/DxilConvergent.cpp",
        "lib/HLSL/DxilEliminateOutputDynamicIndexing.cpp",
        "lib/HLSL/DxilExpandTrigIntrinsics.cpp",
        "lib/HLSL/DxilGenerationPass.cpp",
        "lib/HLSL/DxilLegalizeEvalOperations.cpp",
        "lib/HLSL/DxilLegalizeSampleOffsetPass.cpp",
        "lib/HLSL/DxilLinker.cpp",
        "lib/HLSL/DxilPrecisePropagatePass.cpp",
        "lib/HLSL/DxilPreparePasses.cpp",
        "lib/HLSL/DxilPromoteResourcePasses.cpp",
        "lib/HLSL/DxilPackSignatureElement.cpp",
        "lib/HLSL/DxilPatchShaderRecordBindings.cpp",
        "lib/HLSL/DxilNoops.cpp",
        "lib/HLSL/DxilPreserveAllOutputs.cpp",
        "lib/HLSL/DxilSimpleGVNHoist.cpp",
        "lib/HLSL/DxilSignatureValidation.cpp",
        "lib/HLSL/DxilTargetLowering.cpp",
        "lib/HLSL/DxilTargetTransformInfo.cpp",
        "lib/HLSL/DxilTranslateRawBuffer.cpp",
        "lib/HLSL/DxilExportMap.cpp",
        "lib/HLSL/DxilValidation.cpp",
        "lib/HLSL/DxcOptimizer.cpp",
        "lib/HLSL/HLDeadFunctionElimination.cpp",
        "lib/HLSL/HLExpandStoreIntrinsics.cpp",
        "lib/HLSL/HLLowerUDT.cpp",
        "lib/HLSL/HLMatrixBitcastLowerPass.cpp",
        "lib/HLSL/HLMatrixLowerPass.cpp",
        "lib/HLSL/HLMatrixSubscriptUseReplacer.cpp",
        "lib/HLSL/HLMatrixType.cpp",
        "lib/HLSL/HLMetadataPasses.cpp",
        "lib/HLSL/HLModule.cpp",
        "lib/HLSL/HLOperations.cpp",
        "lib/HLSL/HLOperationLower.cpp",
        "lib/HLSL/HLOperationLowerExtension.cpp",
        "lib/HLSL/HLPreprocess.cpp",
        "lib/HLSL/HLResource.cpp",
        "lib/HLSL/HLSignatureLower.cpp",
        "lib/HLSL/PauseResumePasses.cpp",
        "lib/HLSL/WaveSensitivityAnalysis.cpp",
    ] + glob([
        "lib/HLSL/*.inc",
        "lib/HLSL/*.h",
    ]),
    hdrs = glob([
        "lib/HLSL/*.h",
        "lib/HLSL/*.def",
        "lib/HLSL/*.inc",
    ]),
    copts = [
        "-Wno-switch",
        "-Wno-microsoft-exception-spec",
        "-Wno-inconsistent-missing-override",
        "-Wno-null-dereference",
        "-Wno-unused-variable",
        "-Wno-binding-in-condition",
    ] + select({
        "@toolchains//conditions:ubuntu": [
            "-Wno-misleading-indentation",
        ],
        "//conditions:default": [],
    }) + llvm_copts,
    deps = [
        ":BitReader",
        ":Core",
        ":DXIL",
        ":DxcSupport",
        ":IPA",
        ":Support",
        ":config",
    ],
)

cc_library(
    name = "InstCombine",
    srcs = [
        "lib/Transforms/InstCombine/InstructionCombining.cpp",
        "lib/Transforms/InstCombine/InstCombineAddSub.cpp",
        "lib/Transforms/InstCombine/InstCombineAndOrXor.cpp",
        "lib/Transforms/InstCombine/InstCombineCalls.cpp",
        "lib/Transforms/InstCombine/InstCombineCasts.cpp",
        "lib/Transforms/InstCombine/InstCombineCompares.cpp",
        "lib/Transforms/InstCombine/InstCombineLoadStoreAlloca.cpp",
        "lib/Transforms/InstCombine/InstCombineMulDivRem.cpp",
        "lib/Transforms/InstCombine/InstCombinePHI.cpp",
        "lib/Transforms/InstCombine/InstCombineSelect.cpp",
        "lib/Transforms/InstCombine/InstCombineShifts.cpp",
        "lib/Transforms/InstCombine/InstCombineSimplifyDemanded.cpp",
        "lib/Transforms/InstCombine/InstCombineVectorOps.cpp",
    ] + glob([
        "lib/Transforms/InstCombine/*.inc",
        "lib/Transforms/InstCombine/*.h",
    ]),
    hdrs = glob([
        "lib/Transforms/InstCombine/*.h",
        "lib/Transforms/InstCombine/*.def",
        "lib/Transforms/InstCombine/*.inc",
    ]),
    copts = llvm_copts,
    deps = [
        ":Analysis",
        ":Core",
        ":Support",
        ":TransformUtils",
        ":config",
    ],
)

cc_library(
    name = "IPA",
    srcs = [
        "lib/Analysis/IPA/CallGraph.cpp",
        "lib/Analysis/IPA/CallGraphSCCPass.cpp",
        "lib/Analysis/IPA/CallPrinter.cpp",
        "lib/Analysis/IPA/GlobalsModRef.cpp",
        "lib/Analysis/IPA/IPA.cpp",
        "lib/Analysis/IPA/InlineCost.cpp",
    ] + glob([
        "lib/Analysis/IPA/*.inc",
        "lib/Analysis/IPA/*.h",
    ]),
    hdrs = glob([
        "lib/Analysis/IPA/*.h",
        "lib/Analysis/IPA/*.def",
        "lib/Analysis/IPA/*.inc",
    ]),
    copts = llvm_copts,
    deps = [
        ":Analysis",
        ":Core",
        ":Support",
        ":config",
    ],
)

cc_library(
    name = "IPO",
    srcs = [
        "lib/Transforms/IPO/ArgumentPromotion.cpp",
        "lib/Transforms/IPO/BarrierNoopPass.cpp",
        "lib/Transforms/IPO/ConstantMerge.cpp",
        "lib/Transforms/IPO/DeadArgumentElimination.cpp",
        "lib/Transforms/IPO/ElimAvailExtern.cpp",
        "lib/Transforms/IPO/ExtractGV.cpp",
        "lib/Transforms/IPO/FunctionAttrs.cpp",
        "lib/Transforms/IPO/GlobalDCE.cpp",
        "lib/Transforms/IPO/GlobalOpt.cpp",
        "lib/Transforms/IPO/IPConstantPropagation.cpp",
        "lib/Transforms/IPO/IPO.cpp",
        "lib/Transforms/IPO/InlineAlways.cpp",
        "lib/Transforms/IPO/InlineSimple.cpp",
        "lib/Transforms/IPO/Inliner.cpp",
        "lib/Transforms/IPO/Internalize.cpp",
        "lib/Transforms/IPO/LoopExtractor.cpp",
        "lib/Transforms/IPO/LowerBitSets.cpp",
        "lib/Transforms/IPO/MergeFunctions.cpp",
        "lib/Transforms/IPO/PartialInlining.cpp",
        "lib/Transforms/IPO/PassManagerBuilder.cpp",
        "lib/Transforms/IPO/PruneEH.cpp",
        "lib/Transforms/IPO/StripDeadPrototypes.cpp",
        "lib/Transforms/IPO/StripSymbols.cpp",
    ] + glob([
        "lib/Transforms/IPO/*.inc",
        "include/llvm/Transforms/SampleProfile.h",
        "include/llvm-c/Transforms/IPO.h",
        "include/llvm-c/Transforms/PassManagerBuilder.h",
        "lib/Transforms/IPO/*.h",
    ]),
    hdrs = glob([
        "lib/Transforms/IPO/*.h",
        "lib/Transforms/IPO/*.def",
        "lib/Transforms/IPO/*.inc",
    ]),
    copts = [
        "-Wno-inconsistent-missing-override",
        "-Wno-unused-variable",
    ] + llvm_copts,
    deps = [
        ":Analysis",
        ":Core",
        ":IPA",
        ":InstCombine",
        ":Scalar",
        ":Support",
        ":TransformUtils",
        ":Vectorize",
        ":config",
    ],
)

cc_library(
    name = "IRReader",
    srcs = [
        "lib/IRReader/IRReader.cpp",
    ] + glob([
        "lib/IRReader/*.inc",
        "lib/IRReader/*.h",
    ]),
    hdrs = glob([
        "lib/IRReader/*.h",
        "lib/IRReader/*.def",
        "lib/IRReader/*.inc",
    ]),
    copts = llvm_copts,
    deps = [
        ":AsmParser",
        ":BitReader",
        ":Core",
        ":Support",
        ":config",
    ],
)

cc_library(
    name = "Linker",
    srcs = [
        "lib/Linker/LinkModules.cpp",
    ] + glob([
        "lib/Linker/*.inc",
        "lib/Linker/*.h",
    ]),
    hdrs = glob([
        "lib/Linker/*.h",
        "lib/Linker/*.def",
        "lib/Linker/*.inc",
    ]),
    copts = llvm_copts,
    deps = [
        ":Core",
        ":Support",
        ":TransformUtils",
        ":config",
    ],
)

cc_library(
    name = "LTO",
    srcs = [
        "lib/LTO/LTOModule.cpp",
        "lib/LTO/LTOCodeGenerator.cpp",
    ] + glob([
        "lib/LTO/*.inc",
        "lib/LTO/*.h",
    ]),
    hdrs = glob([
        "lib/LTO/*.h",
        "lib/LTO/*.def",
        "lib/LTO/*.inc",
    ]),
    copts = llvm_copts,
    deps = [
        ":Analysis",
        ":BitReader",
        ":BitWriter",
        ":Core",
        ":IPA",
        ":IPO",
        ":InstCombine",
        ":Linker",
        ":Scalar",
        ":Support",
        ":Target",
        ":config",
    ],
)

cc_library(
    name = "MSSupport",
    srcs = [
        "lib/MSSupport/MSFileSystemImpl.cpp",
    ],
    hdrs = [
        "include/llvm/Support/MSFileSystem.h",
    ],
    copts = [
        "-Wno-macro-redefined",
    ] + llvm_copts,
    deps = [
        ":config",
        "@zlib",
    ],
)

cc_library(
    name = "Option",
    srcs = [
        "lib/Option/Arg.cpp",
        "lib/Option/ArgList.cpp",
        "lib/Option/Option.cpp",
        "lib/Option/OptTable.cpp",
    ] + glob([
        "lib/Option/*.inc",
        "lib/Option/*.h",
    ]),
    hdrs = glob([
        "lib/Option/*.h",
        "lib/Option/*.def",
        "lib/Option/*.inc",
    ]),
    copts = llvm_copts,
    deps = [
        ":Support",
        ":config",
    ],
)

cc_library(
    name = "Passes",
    srcs = [
        "lib/Passes/PassBuilder.cpp",
    ] + glob([
        "lib/Passes/*.inc",
        "lib/Passes/*.h",
    ]),
    hdrs = glob([
        "lib/Passes/*.h",
        "lib/Passes/*.def",
        "lib/Passes/*.inc",
    ]),
    copts = llvm_copts,
    deps = [
        ":Analysis",
        ":Core",
        ":IPA",
        ":IPO",
        ":InstCombine",
        ":Scalar",
        ":Support",
        ":TransformUtils",
        ":Vectorize",
        ":config",
    ],
)

cc_library(
    name = "PassPrinters",
    srcs = [
        "lib/PassPrinters/PassPrinters.cpp",
    ] + glob([
        "lib/PassPrinters/*.inc",
        "lib/PassPrinters/*.h",
    ]),
    hdrs = glob([
        "lib/PassPrinters/*.h",
        "lib/PassPrinters/*.def",
        "lib/PassPrinters/*.inc",
    ]),
    copts = llvm_copts,
    deps = [
        ":Analysis",
        ":Core",
        ":IPA",
        ":IPO",
        ":InstCombine",
        ":Passes",
        ":Scalar",
        ":Support",
        ":TransformUtils",
        ":Vectorize",
        ":config",
    ],
)

cc_library(
    name = "ProfileData",
    srcs = [
        "lib/ProfileData/InstrProf.cpp",
        "lib/ProfileData/InstrProfReader.cpp",
        "lib/ProfileData/InstrProfWriter.cpp",
        "lib/ProfileData/CoverageMapping.cpp",
        "lib/ProfileData/CoverageMappingWriter.cpp",
        "lib/ProfileData/CoverageMappingReader.cpp",
        "lib/ProfileData/SampleProf.cpp",
        "lib/ProfileData/SampleProfReader.cpp",
        "lib/ProfileData/SampleProfWriter.cpp",
    ] + glob([
        "lib/ProfileData/*.inc",
        "lib/ProfileData/*.h",
    ]),
    hdrs = glob([
        "lib/ProfileData/*.h",
        "lib/ProfileData/*.def",
        "lib/ProfileData/*.inc",
    ]),
    copts = llvm_copts,
    deps = [
        ":Core",
        ":Support",
        ":config",
    ],
)

# LLVMScalarOpts
cc_library(
    name = "Scalar",
    srcs = [
        "lib/Transforms/Scalar/ADCE.cpp",
        "lib/Transforms/Scalar/AlignmentFromAssumptions.cpp",
        "lib/Transforms/Scalar/BDCE.cpp",
        "lib/Transforms/Scalar/ConstantHoisting.cpp",
        "lib/Transforms/Scalar/ConstantProp.cpp",
        "lib/Transforms/Scalar/CorrelatedValuePropagation.cpp",
        "lib/Transforms/Scalar/DCE.cpp",
        "lib/Transforms/Scalar/DeadStoreElimination.cpp",
        "lib/Transforms/Scalar/EarlyCSE.cpp",
        "lib/Transforms/Scalar/FlattenCFGPass.cpp",
        "lib/Transforms/Scalar/Float2Int.cpp",
        "lib/Transforms/Scalar/GVN.cpp",
        "lib/Transforms/Scalar/HoistConstantArray.cpp",
        "lib/Transforms/Scalar/InductiveRangeCheckElimination.cpp",
        "lib/Transforms/Scalar/IndVarSimplify.cpp",
        "lib/Transforms/Scalar/JumpThreading.cpp",
        "lib/Transforms/Scalar/LICM.cpp",
        "lib/Transforms/Scalar/LoadCombine.cpp",
        "lib/Transforms/Scalar/LoopDeletion.cpp",
        "lib/Transforms/Scalar/LoopDistribute.cpp",
        "lib/Transforms/Scalar/LoopIdiomRecognize.cpp",
        "lib/Transforms/Scalar/LoopInstSimplify.cpp",
        "lib/Transforms/Scalar/LoopInterchange.cpp",
        "lib/Transforms/Scalar/LoopRerollPass.cpp",
        "lib/Transforms/Scalar/LoopRotation.cpp",
        "lib/Transforms/Scalar/LoopStrengthReduce.cpp",
        "lib/Transforms/Scalar/LoopUnrollPass.cpp",
        "lib/Transforms/Scalar/LoopUnswitch.cpp",
        "lib/Transforms/Scalar/LowerAtomic.cpp",
        "lib/Transforms/Scalar/LowerExpectIntrinsic.cpp",
        "lib/Transforms/Scalar/LowerTypePasses.cpp",
        "lib/Transforms/Scalar/MemCpyOptimizer.cpp",
        "lib/Transforms/Scalar/MergedLoadStoreMotion.cpp",
        "lib/Transforms/Scalar/NaryReassociate.cpp",
        "lib/Transforms/Scalar/PartiallyInlineLibCalls.cpp",
        "lib/Transforms/Scalar/PlaceSafepoints.cpp",
        "lib/Transforms/Scalar/Reassociate.cpp",
        "lib/Transforms/Scalar/Reg2Mem.cpp",
        "lib/Transforms/Scalar/Reg2MemHLSL.cpp",
        "lib/Transforms/Scalar/RewriteStatepointsForGC.cpp",
        "lib/Transforms/Scalar/SCCP.cpp",
        "lib/Transforms/Scalar/SROA.cpp",
        "lib/Transforms/Scalar/SampleProfile.cpp",
        "lib/Transforms/Scalar/Scalar.cpp",
        "lib/Transforms/Scalar/ScalarReplAggregates.cpp",
        "lib/Transforms/Scalar/ScalarReplAggregatesHLSL.cpp",
        "lib/Transforms/Scalar/DxilLoopUnroll.cpp",
        "lib/Transforms/Scalar/DxilRemoveDeadBlocks.cpp",
        "lib/Transforms/Scalar/DxilEraseDeadRegion.cpp",
        "lib/Transforms/Scalar/DxilFixConstArrayInitializer.cpp",
        "lib/Transforms/Scalar/DxilEliminateVector.cpp",
        "lib/Transforms/Scalar/Scalarizer.cpp",
        "lib/Transforms/Scalar/SeparateConstOffsetFromGEP.cpp",
        "lib/Transforms/Scalar/SimplifyCFGPass.cpp",
        "lib/Transforms/Scalar/Sink.cpp",
        "lib/Transforms/Scalar/SpeculativeExecution.cpp",
        "lib/Transforms/Scalar/StraightLineStrengthReduce.cpp",
        "lib/Transforms/Scalar/StructurizeCFG.cpp",
        "lib/Transforms/Scalar/TailRecursionElimination.cpp",
    ] + glob([
        "lib/Transforms/Scalar/*.inc",
        "include/llvm-c/Transforms/Scalar.h",
        "include/llvm/Transforms/Scalar.h",
        "include/llvm/Target/TargetMachine.h",
        "lib/Transforms/Scalar/*.h",
    ]),
    hdrs = glob([
        "lib/Transforms/Scalar/*.h",
        "lib/Transforms/Scalar/*.def",
        "lib/Transforms/Scalar/*.inc",
        "include/llvm/Transforms/IPO.h",
        "include/llvm/Transforms/IPO/SCCP.h",
    ]),
    copts = [
        "-Wno-switch",
        "-Wno-inconsistent-missing-override",
        "-Wno-binding-in-condition",
        "-Wno-gnu-include-next",
    ] + select({
        "@toolchains//conditions:ubuntu": [
            "-Wno-misleading-indentation",
            "-Wno-int-in-bool-context",
        ],
        "//conditions:default": [],
    }) + llvm_copts,
    deps = [
        ":Analysis",
        ":Core",
        ":InstCombine",
        ":ProfileData",
        ":Support",
        ":Target",
        ":TransformUtils",
        ":config",
    ],
)

cc_library(
    name = "Support",
    srcs = [
        "lib/Support/APFloat.cpp",
        "lib/Support/APInt.cpp",
        "lib/Support/APSInt.cpp",
        "lib/Support/ARMBuildAttrs.cpp",
        "lib/Support/ARMWinEH.cpp",
        "lib/Support/Allocator.cpp",
        "lib/Support/Atomic.cpp",
        "lib/Support/BlockFrequency.cpp",
        "lib/Support/BranchProbability.cpp",
        "lib/Support/COM.cpp",
        "lib/Support/CommandLine.cpp",
        "lib/Support/Compression.cpp",
        "lib/Support/ConvertUTF.c",
        "lib/Support/ConvertUTFWrapper.cpp",
        "lib/Support/CrashRecoveryContext.cpp",
        "lib/Support/DAGDeltaAlgorithm.cpp",
        "lib/Support/DataExtractor.cpp",
        "lib/Support/DataStream.cpp",
        "lib/Support/Debug.cpp",
        "lib/Support/DeltaAlgorithm.cpp",
        "lib/Support/Dwarf.cpp",
        "lib/Support/Errno.cpp",
        "lib/Support/ErrorHandling.cpp",
        "lib/Support/FileOutputBuffer.cpp",
        "lib/Support/FileUtilities.cpp",
        "lib/Support/FoldingSet.cpp",
        "lib/Support/FormattedStream.cpp",
        "lib/Support/GraphWriter.cpp",
        "lib/Support/Hashing.cpp",
        "lib/Support/Host.cpp",
        "lib/Support/IntEqClasses.cpp",
        "lib/Support/IntervalMap.cpp",
        "lib/Support/IntrusiveRefCntPtr.cpp",
        "lib/Support/LEB128.cpp",
        "lib/Support/LineIterator.cpp",
        "lib/Support/Locale.cpp",
        "lib/Support/LockFileManager.cpp",
        "lib/Support/MD5.cpp",
        "lib/Support/MSFileSystemBasic.cpp",
        "lib/Support/ManagedStatic.cpp",
        "lib/Support/MathExtras.cpp",
        "lib/Support/Memory.cpp",
        "lib/Support/MemoryBuffer.cpp",
        "lib/Support/MemoryObject.cpp",
        "lib/Support/Mutex.cpp",
        "lib/Support/Options.cpp",
        "lib/Support/Path.cpp",
        "lib/Support/PrettyStackTrace.cpp",
        "lib/Support/Process.cpp",
        "lib/Support/Program.cpp",
        "lib/Support/RWMutex.cpp",
        "lib/Support/RandomNumberGenerator.cpp",
        "lib/Support/Regex.cpp",
        "lib/Support/ScaledNumber.cpp",
        "lib/Support/SearchForAddressOfSpecialSymbol.cpp",
        "lib/Support/Signals.cpp",
        "lib/Support/SmallPtrSet.cpp",
        "lib/Support/SmallVector.cpp",
        "lib/Support/SourceMgr.cpp",
        "lib/Support/SpecialCaseList.cpp",
        "lib/Support/Statistic.cpp",
        "lib/Support/StreamingMemoryObject.cpp",
        "lib/Support/StringExtras.cpp",
        "lib/Support/StringMap.cpp",
        "lib/Support/StringPool.cpp",
        "lib/Support/StringRef.cpp",
        "lib/Support/StringSaver.cpp",
        "lib/Support/SystemUtils.cpp",
        "lib/Support/TargetParser.cpp",
        "lib/Support/TargetRegistry.cpp",
        "lib/Support/ThreadLocal.cpp",
        "lib/Support/Threading.cpp",
        "lib/Support/TimeValue.cpp",
        "lib/Support/Timer.cpp",
        "lib/Support/ToolOutputFile.cpp",
        "lib/Support/Triple.cpp",
        "lib/Support/Twine.cpp",
        "lib/Support/Unicode.cpp",
        "lib/Support/Valgrind.cpp",
        "lib/Support/Watchdog.cpp",
        "lib/Support/YAMLParser.cpp",
        "lib/Support/YAMLTraits.cpp",
        "lib/Support/assert.cpp",
        "lib/Support/circular_raw_ostream.cpp",
        "lib/Support/raw_os_ostream.cpp",
        "lib/Support/raw_ostream.cpp",
        "lib/Support/regcomp.c",
        "lib/Support/regerror.c",
        "lib/Support/regexec.c",
        "lib/Support/regfree.c",
        "lib/Support/regmalloc.cpp",
        "lib/Support/regstrlcpy.c",
    ] + glob([
        "lib/Support/*.inc",
        "include/llvm-c/*.h",
        "include/llvm/CodeGen/MachineValueType.h",
        "include/llvm/Support/COFF.h",
        "include/llvm/Support/MachO.h",
        "lib/Support/*.h",
    ]) + select({
        "@toolchains//conditions:windows": glob([
            "lib/Support/Windows/*.inc",
            "lib/Support/Windows/*.h",
        ]),
        "@toolchains//conditions:linux": glob([
            "lib/Support/Unix/*.inc",
            "lib/Support/Unix/*.h",
        ]),
    }),
    hdrs = glob([
        "lib/Support/*.h",
        "lib/Support/*.def",
        "lib/Support/*.inc",
        "include/llvm/ADT/*.h",
        "include/llvm/Support/ELFRelocs/*.def",
        "include/llvm/Support/WasmRelocs/*.def",
    ]) + [
        "include/llvm/Support/VCSRevision.h",
        "include/llvm/Support/DataTypes.h",
    ],
    copts = [
        "-Wno-invalid-noreturn",
        "-Wno-macro-redefined",
        "-Wno-unused-function",
    ] + llvm_copts,
    defines = select({
        "@toolchains//conditions:ubuntu": [
            "__STDC_LIMIT_MACROS",
            "__STDC_CONSTANT_MACROS",
        ],
        "//conditions:default": [
        ],
    }),
    deps = [
        ":config",
        "@zlib",
    ],
)

cc_library(
    name = "TableGen",
    srcs = [
        "lib/TableGen/Error.cpp",
        "lib/TableGen/Main.cpp",
        "lib/TableGen/Record.cpp",
        "lib/TableGen/SetTheory.cpp",
        "lib/TableGen/StringMatcher.cpp",
        "lib/TableGen/TableGenBackend.cpp",
        "lib/TableGen/TGLexer.cpp",
        "lib/TableGen/TGParser.cpp",
    ] + glob([
        "lib/TableGen/*.inc",
        "include/llvm/CodeGen/*.h",
        "lib/TableGen/*.h",
    ]),
    hdrs = glob([
        "lib/TableGen/*.h",
        "lib/TableGen/*.def",
        "lib/TableGen/*.inc",
        "include/llvm/Target/*.def",
    ]),
    copts = llvm_copts,
    deps = [
        ":Support",
        ":config",
    ],
)

cc_library(
    name = "Target",
    srcs = [
        "lib/Target/Target.cpp",
        "lib/Target/TargetIntrinsicInfo.cpp",
        "lib/Target/TargetLoweringObjectFile.cpp",
        "lib/Target/TargetMachine.cpp",
        "lib/Target/TargetMachineC.cpp",
        "lib/Target/TargetRecip.cpp",
        "lib/Target/TargetSubtargetInfo.cpp",
    ] + glob([
        "lib/Target/*.inc",
        "include/llvm/CodeGen/*.h",
        "include/llvm-c/Initialization.h",
        "include/llvm-c/Target.h",
        "lib/Target/*.h",
    ]),
    hdrs = glob([
        "lib/Target/*.h",
        "lib/Target/*.def",
        "lib/Target/*.inc",
        "include/llvm/CodeGen/*.def",
        "include/llvm/CodeGen/*.inc",
    ]),
    copts = llvm_copts,
    deps = [
        ":Analysis",
        ":Core",
        ":Support",
        ":config",
    ],
)

cc_library(
    name = "TransformUtils",
    srcs = [
        "lib/Transforms/Utils/ASanStackFrameLayout.cpp",
        "lib/Transforms/Utils/AddDiscriminators.cpp",
        "lib/Transforms/Utils/BasicBlockUtils.cpp",
        "lib/Transforms/Utils/BreakCriticalEdges.cpp",
        "lib/Transforms/Utils/BuildLibCalls.cpp",
        "lib/Transforms/Utils/BypassSlowDivision.cpp",
        "lib/Transforms/Utils/CloneFunction.cpp",
        "lib/Transforms/Utils/CloneModule.cpp",
        "lib/Transforms/Utils/CmpInstAnalysis.cpp",
        "lib/Transforms/Utils/CodeExtractor.cpp",
        "lib/Transforms/Utils/CtorUtils.cpp",
        "lib/Transforms/Utils/DemoteRegToStack.cpp",
        "lib/Transforms/Utils/FlattenCFG.cpp",
        "lib/Transforms/Utils/GlobalStatus.cpp",
        "lib/Transforms/Utils/InlineFunction.cpp",
        "lib/Transforms/Utils/InstructionNamer.cpp",
        "lib/Transforms/Utils/IntegerDivision.cpp",
        "lib/Transforms/Utils/LCSSA.cpp",
        "lib/Transforms/Utils/Local.cpp",
        "lib/Transforms/Utils/LoopSimplify.cpp",
        "lib/Transforms/Utils/LoopUnroll.cpp",
        "lib/Transforms/Utils/LoopUnrollRuntime.cpp",
        "lib/Transforms/Utils/LoopUtils.cpp",
        "lib/Transforms/Utils/LoopVersioning.cpp",
        "lib/Transforms/Utils/LowerInvoke.cpp",
        "lib/Transforms/Utils/LowerSwitch.cpp",
        "lib/Transforms/Utils/Mem2Reg.cpp",
        "lib/Transforms/Utils/MetaRenamer.cpp",
        "lib/Transforms/Utils/ModuleUtils.cpp",
        "lib/Transforms/Utils/PromoteMemoryToRegister.cpp",
        "lib/Transforms/Utils/SSAUpdater.cpp",
        "lib/Transforms/Utils/SimplifyCFG.cpp",
        "lib/Transforms/Utils/SimplifyIndVar.cpp",
        "lib/Transforms/Utils/SimplifyInstructions.cpp",
        "lib/Transforms/Utils/SimplifyLibCalls.cpp",
        "lib/Transforms/Utils/SymbolRewriter.cpp",
        "lib/Transforms/Utils/UnifyFunctionExitNodes.cpp",
        "lib/Transforms/Utils/Utils.cpp",
        "lib/Transforms/Utils/ValueMapper.cpp",
    ] + glob([
        "lib/Transforms/Utils/*.inc",
        "include/llvm/Transforms/IPO.h",
        "include/llvm/Transforms/Scalar.h",
        "lib/Transforms/Utils/*.h",
    ]),
    hdrs = glob([
        "lib/Transforms/Utils/*.h",
        "lib/Transforms/Utils/*.def",
        "lib/Transforms/Utils/*.inc",
    ]),
    copts = llvm_copts,
    deps = [
        ":Analysis",
        ":Core",
        ":IPA",
        ":Support",
        ":config",
    ],
)

cc_library(
    name = "Vectorize",
    srcs = [
        "lib/Transforms/Vectorize/Vectorize.cpp",
    ] + glob([
        "lib/Transforms/Vectorize/*.inc",
        "include/llvm-c/Transforms/Vectorize.h",
        "lib/Transforms/Vectorize/*.h",
    ]),
    hdrs = glob([
        "lib/Transforms/Vectorize/*.h",
        "lib/Transforms/Vectorize/*.def",
        "lib/Transforms/Vectorize/*.inc",
        "include/llvm/Transforms/Vectorize.h",
    ]),
    copts = [
        "-Wno-switch",
    ] + llvm_copts,
    deps = [
        ":Analysis",
        ":Core",
        ":Scalar",
        ":Support",
        ":TransformUtils",
        ":config",
    ],
)
