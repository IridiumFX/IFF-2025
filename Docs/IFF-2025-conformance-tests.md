IFF-2025 Conformance Test Matrix
=================================

This document defines the complete set of conformance tests for IFF-2025
parser and generator implementations. Tests are numbered globally and
organized by functional area.

Tests marked **[COVERED]** have existing implementations. All others are new.


Part A: Read Path (Parser)
==========================


A1. Bootstrapping and Mode Selection
-------------------------------------

### A1.1 IFF-85 Auto-Detection

| #  | Test | Description |
|----|------|-------------|
| R1 | iff85_form_locks_session | First tag is FORM. Parser enters IFF-85 mode, `iff85_locked == 1`. Session completes. **[COVERED: test 1]** |
| R2 | iff85_list_locks_session | First tag is LIST. Parser enters IFF-85 mode, `iff85_locked == 1`. |
| R3 | iff85_cat_locks_session | First tag is CAT. Parser enters IFF-85 mode, `iff85_locked == 1`. |
| R4 | iff85_rejects_midstream_directive | IFF-85 locked session encounters ' VER' directive inside FORM. Parse fails. |
| R5 | iff85_allows_filler | IFF-85 locked session encounters '    ' (filler) inside FORM. Filler is silently consumed. Parse succeeds. |
| R6 | iff85_default_flags | IFF-85 mode uses default flags: 4-byte tags, 32-bit BE signed sizes, blobbed, padding enabled, no sharding. Verify all flag fields are zero. |

### A1.2 IFF-2025 Explicit Activation

| #  | Test | Description |
|----|------|-------------|
| R7 | iff2025_header_activates | First tag is ' IFF' with version=40. `iff85_locked == 0`. Session completes. **[COVERED: test 2]** |
| R8 | iff2025_header_default_flags | ' IFF' with all-zero flags produces identical behavior to IFF-85 defaults, but `iff85_locked == 0`. Directives are accepted. |
| R9 | iff2025_version_zero_locks_85 | ' IFF' with version=0 activates IFF-85 mode. `iff85_locked == 1`. |
| R10 | iff2025_unknown_version_fails | ' IFF' with version=99. Parse fails (unrecognized version). |


A2. Header Flag Processing
---------------------------

### A2.1 Sizing Flags

| #  | Test | Description |
|----|------|-------------|
| R11 | sizing_32_default | ' IFF' with sizing=0 (32-bit). FORM chunk sizes read as 4-byte BE. Parse succeeds. |
| R12 | sizing_64 | ' IFF' with sizing=1 (64-bit). FORM and chunk sizes read as 8-byte fields. Parse succeeds. |
| R13 | sizing_16 | ' IFF' with sizing=255 (16-bit). FORM and chunk sizes read as 2-byte fields. Parse succeeds. |

### A2.2 Tag Sizing Flags

| #  | Test | Description |
|----|------|-------------|
| R14 | tag_sizing_4_default | ' IFF' with tag_sizing=0 (4-byte). Tags read as 4 bytes, right-padded to canonical 16. Parse succeeds. |
| R15 | tag_sizing_8 | ' IFF' with tag_sizing=1 (8-byte). Tags read as 8 bytes. FORM and chunk tags correct. Parse succeeds. |
| R16 | tag_sizing_16 | ' IFF' with tag_sizing=2 (16-byte). Tags read as 16 bytes. Parse succeeds. |

### A2.3 Endianness and Signedness

| #  | Test | Description |
|----|------|-------------|
| R17 | little_endian_sizes | ' IFF' with typing LITTLE_ENDIAN. All size fields read as little-endian. Build image with LE sizes via TestBuilder. Parse succeeds. |
| R18 | unsigned_sizes | ' IFF' with typing UNSIGNED_SIZES. Sizes interpreted as unsigned. Parse succeeds. |
| R19 | le_unsigned_combined | ' IFF' with typing = LITTLE_ENDIAN | UNSIGNED_SIZES. Both flags active. Parse succeeds. |

### A2.4 Operating Mode

| #  | Test | Description |
|----|------|-------------|
| R20 | progressive_form | ' IFF' with operating=PROGRESSIVE. FORM has no size field, terminated by ' END'. Parse succeeds. **[COVERED: test 3]** |
| R21 | progressive_list | Progressive LIST with PROP and nested FORM, each terminated by ' END'. Parse succeeds. **[COVERED: test 8]** |
| R22 | progressive_cat | Progressive CAT with two nested FORMs, each terminated by ' END'. Parse succeeds. |

### A2.5 Structuring Flags

| #  | Test | Description |
|----|------|-------------|
| R23 | no_padding_flag | ' IFF' with structuring NO_PADDING. Odd-size chunk (5 bytes) has no padding byte after it. Next chunk follows immediately. Parse succeeds. |
| R24 | default_padding | Default flags. Odd-size chunk (5 bytes) followed by 1 pad byte. Parser skips pad. Parse succeeds. |
| R25 | strict_containers_matching | ' IFF' with STRICT_CONTAINERS. LIST type=ILBM contains FORM type=ILBM. Types match. Parse succeeds. |
| R26 | strict_containers_mismatch | ' IFF' with STRICT_CONTAINERS. LIST type=ILBM contains FORM type=8SVX. Types don't match. Parse fails. |
| R27 | strict_containers_wildcard_parent | ' IFF' with STRICT_CONTAINERS. CAT type="    " contains FORM type=ILBM. Wildcard parent allows any child. Parse succeeds. |


A3. Container Parsing
-----------------------

### A3.1 FORM

| #  | Test | Description |
|----|------|-------------|
| R28 | form_basic_blobbed | Blobbed FORM with two data chunks. Boundary matches declared size. Parse succeeds. **[COVERED: test 1]** |
| R29 | form_empty | FORM with no content chunks (size = type_tag_length only). Parse succeeds. |
| R30 | form_nested_in_list | FORM nested inside LIST. Parse succeeds. **[COVERED: test 5]** |
| R31 | form_nested_in_cat | FORM nested inside CAT. Parse succeeds. **[COVERED: test 6]** |

### A3.2 LIST

| #  | Test | Description |
|----|------|-------------|
| R32 | list_with_prop_and_form | LIST containing PROP then FORM. Parse succeeds. **[COVERED: test 5]** |
| R33 | list_with_multiple_props | LIST containing two PROPs (same type), second overrides first. Decoder sees second PROP's chunks. |
| R34 | list_wildcard_type | LIST with type "    " containing FORMs of different types. Parse succeeds. |
| R35 | list_nested_in_list | LIST nested inside LIST. Parse succeeds. **[COVERED: test 7]** |
| R36 | list_empty | LIST with no child containers (size = type_tag_length). Parse succeeds. |

### A3.3 CAT

| #  | Test | Description |
|----|------|-------------|
| R37 | cat_with_forms | CAT with two differently-typed FORMs. Parse succeeds. **[COVERED: test 6]** |
| R38 | cat_with_mixed_containers | CAT containing FORM, LIST, and nested CAT. Parse succeeds. |
| R39 | cat_empty | CAT with no child containers. Parse succeeds. |

### A3.4 PROP

| #  | Test | Description |
|----|------|-------------|
| R40 | prop_chunks_stored | PROP inside LIST stores chunks. FormDecoder's FindProp retrieves them. **[COVERED: test 17]** |
| R41 | prop_wildcard_type | PROP with type "    " in LIST. Properties available to all nested FORMs regardless of type. |
| R42 | prop_specific_type | PROP with type=ILBM in LIST. Properties available only to ILBM FORMs, not 8SVX. |
| R43 | prop_fallback_resolution | LIST with PROP("    ") and PROP("ILBM"). ILBM FORM sees ILBM PROP first; non-ILBM FORM falls back to wildcard. |
| R44 | prop_empty | PROP with no chunks. Parse succeeds. No properties stored. |
| R45 | prop_between_forms | PROP appearing between two FORMs in a LIST. Second FORM sees updated properties. |


A4. Chunk and Padding
-----------------------

| #  | Test | Description |
|----|------|-------------|
| R46 | chunk_even_size | Chunk with even size (10 bytes). No padding byte. Parse succeeds. |
| R47 | chunk_odd_size_padded | Chunk with odd size (5 bytes). Parser reads 1 pad byte. Parse succeeds. |
| R48 | chunk_zero_size | Chunk with size=0 and no payload. Parse succeeds. |
| R49 | chunk_odd_no_padding_flag | NO_PADDING flag set. Chunk with odd size (5 bytes). No pad byte read. Next tag immediately follows. Parse succeeds. |
| R50 | chunk_large_payload | Chunk with 1024-byte payload. Data read correctly. Parse succeeds. |
| R51 | chunk_unknown_tag_skipped | Chunk with unregistered tag in a FORM. Parser reads and skips it (forward compatibility). Parse succeeds. |


A5. Directive Handling
-----------------------

### A5.1 Core Directives

| #  | Test | Description |
|----|------|-------------|
| R52 | end_terminates_progressive | ' END' with size=0 terminates progressive container. Scope closed correctly. **[COVERED: test 3]** |
| R53 | end_nonzero_size_fails | ' END' with size != 0. Parse fails. |
| R54 | filler_skipped_default | '    ' directive (no SHARDING flag) inside FORM. Parser reads and skips it. Parse succeeds. |
| R55 | unknown_directive_skipped | Unknown directive ' FOO' inside FORM (IFF-2025 mode). Parser reads and skips it. Parse succeeds. |

### A5.2 ' DEF' and ' REF'

| #  | Test | Description |
|----|------|-------------|
| R56 | def_directive_skipped | ' DEF' at root level consumed silently. Parse succeeds. **[COVERED: test 12]** |
| R57 | ref_optional_no_resolver | ' REF' with id_size=0 (optional) and no resolver registered. Silently skipped. Parse succeeds. **[COVERED: test 14]** |
| R58 | ref_mandatory_no_resolver | ' REF' with id_size>0 (mandatory) and no resolver registered. Parse fails (unresolvable). |

### A5.3 ' VER' and ' REV'

| #  | Test | Description |
|----|------|-------------|
| R59 | ver_directive_skipped | ' VER' inside FORM in IFF-2025 mode. Read and skipped. Parse succeeds. |
| R60 | rev_directive_skipped | ' REV' inside FORM in IFF-2025 mode. Read and skipped. Parse succeeds. |

### A5.4 Mid-Stream ' IFF'

| #  | Test | Description |
|----|------|-------------|
| R61 | midstream_iff_narrows_sizing | ' IFF' inside LIST narrows sizing from 32-bit to 16-bit. Subsequent chunks use 16-bit sizes. Parse succeeds. **[COVERED: test 18]** |
| R62 | midstream_iff_narrows_tags | ' IFF' inside LIST narrows tag_sizing from 8-byte to 4-byte. Subsequent tags use 4-byte width. Parse succeeds. |
| R63 | midstream_iff_enables_no_padding | ' IFF' inside FORM enables NO_PADDING. Subsequent odd chunks have no pad byte. Parse succeeds. |
| R64 | midstream_iff_enables_sharding | ' IFF' inside FORM enables SHARDING. Subsequent '    ' chunks dispatched to decoder as shards. Parse succeeds. |


A6. Scope Guards (Mid-Stream ' IFF' Rejection)
------------------------------------------------

| #  | Test | Description |
|----|------|-------------|
| R65 | guard_sizing_widening | ' IFF' widens sizing from 16-bit to 32-bit inside a scope. Guard fires. Parse fails. |
| R66 | guard_tag_widening | ' IFF' widens tag_sizing from 4-byte to 8-byte inside a scope. Guard fires. Parse fails. |
| R67 | guard_blobbed_to_progressive | ' IFF' switches from blobbed to progressive inside a blobbed parent. Guard fires. Parse fails. **[COVERED: test 19]** |
| R68 | guard_progressive_to_blobbed_ok | ' IFF' switches from progressive to blobbed inside a progressive parent. Guard does NOT fire (narrowing is allowed). Parse succeeds. |
| R69 | guard_sizing_narrowing_ok | ' IFF' narrows sizing from 64-bit to 32-bit. Guard does NOT fire. Parse succeeds. |
| R70 | guard_tag_narrowing_ok | ' IFF' narrows tag_sizing from 16-byte to 8-byte. Guard does NOT fire. Parse succeeds. |


A7. Decoder Lifecycle
-----------------------

### A7.1 ChunkDecoder

| #  | Test | Description |
|----|------|-------------|
| R71 | chunk_decoder_lifecycle | Registered ChunkDecoder for BMHD. begin_decode, process_shard, end_decode all called. ContextualData produced. **[COVERED: test 16]** |
| R72 | chunk_decoder_no_registration | FORM chunk with no registered decoder. Chunk read and skipped. Parse succeeds. |
| R73 | chunk_decoder_in_prop | ChunkDecoder processes chunk inside PROP. Result stored as property. |

### A7.2 FormDecoder

| #  | Test | Description |
|----|------|-------------|
| R74 | form_decoder_lifecycle | Registered FormDecoder for ILBM. begin_decode, process_chunk, end_decode all called. Final entity produced. **[COVERED: test 15]** |
| R75 | form_decoder_nested | FormDecoder receives nested FORM via process_nested_form callback. Entity from inner FORM propagated. |
| R76 | form_decoder_no_registration | FORM with no registered decoder. Chunks read and skipped. Parse succeeds. |
| R77 | form_decoder_find_prop | FormDecoder calls FindProp in begin_decode. PROP data from enclosing LIST retrieved. **[COVERED: test 17]** |
| R78 | form_decoder_error_propagation | FormDecoder's begin_decode returns 0. Parse fails. |


A8. Sharding
--------------

| #  | Test | Description |
|----|------|-------------|
| R79 | shard_single | SHARDING enabled. Chunk BMHD followed by one '    ' shard. ChunkDecoder's process_shard called twice (once for chunk, once for shard). |
| R80 | shard_multiple | SHARDING enabled. Chunk followed by three '    ' shards. process_shard called four times total. |
| R81 | shard_flush_on_next_chunk | After shard sequence, next non-shard chunk triggers decoder end_decode (flush). |
| R82 | shard_flush_on_scope_exit | After shard sequence, EndForm triggers decoder end_decode (flush). |
| R83 | shard_flush_on_nested_container | After shard sequence, nested FORM triggers decoder end_decode (flush). |
| R84 | shard_no_pending_decoder | SHARDING enabled. '    ' shard with no pending decoder. Silently consumed. Parse succeeds. |
| R85 | shard_disabled_acts_as_filler | SHARDING NOT enabled. '    ' chunk treated as filler and skipped. No decoder interaction. |


A9. Checksum Verification
---------------------------

| #  | Test | Description |
|----|------|-------------|
| R86 | checksum_roundtrip_basic | ' CHK' and ' SUM' around a chunk. Checksum computed correctly. Parse succeeds. **[COVERED: test 20]** |
| R87 | checksum_mismatch_fails | ' SUM' contains wrong expected checksum. Computed != expected. Parse fails. |
| R88 | checksum_empty_span | ' CHK' immediately followed by ' SUM' with no data between. Checksum of empty byte sequence verified. Parse succeeds. |
| R89 | checksum_nested_spans | Two CHK..SUM pairs nested (inner span inside outer span). Both verified independently. LIFO order. Parse succeeds. |
| R90 | checksum_multiple_algorithms | ' CHK' lists two algorithm IDs. Both checksums computed and verified in ' SUM'. Parse succeeds. |
| R91 | checksum_partial_algorithm_support | ' CHK' lists 3 algorithms, only 2 registered. Registered ones verified, unregistered skipped. Parse succeeds. |
| R92 | checksum_lrc | LRC-ISO-1155 algorithm: known input produces expected 8-bit XOR checksum. |
| R93 | checksum_rfc1071 | RFC-1071 algorithm: known input produces expected 16-bit one's complement sum. |
| R94 | checksum_crc32c | CRC-32C algorithm: known input produces expected 32-bit Castagnoli CRC. |
| R95 | checksum_crc64ecma | CRC-64/ECMA algorithm: known input produces expected 64-bit ECMA-182 CRC. |


A10. Boundary and Size Validation
-----------------------------------

| #  | Test | Description |
|----|------|-------------|
| R96 | boundary_exact_match | FORM declared size exactly matches total content read. Parse succeeds. |
| R97 | boundary_overrun | FORM declared size is 10 but content is 20 bytes. Parser reads past boundary. Parse fails. |
| R98 | boundary_underrun | FORM declared size is 40 but content is only 20 bytes. EOF within container. Parse fails. |
| R99 | boundary_progressive_unbounded | Progressive FORM has limit=0 (unbounded). Parser reads until ' END'. No boundary check on size. |
| R100 | padding_included_in_boundary | Blobbed FORM with odd-size chunk. Padding byte counted in parent's boundary level. Exact match. |


A11. Multiple Top-Level Forms and EOF
---------------------------------------

| #  | Test | Description |
|----|------|-------------|
| R101 | multiple_top_forms | Two consecutive top-level FORMs after ' IFF'. Both parsed. Session completes. **[COVERED: test 13]** |
| R102 | eof_after_single_form | Single FORM at top level. EOF after FORM. Session state = Complete. |
| R103 | eof_mid_chunk | EOF in the middle of reading a chunk's payload. Parse fails. |
| R104 | eof_mid_tag | EOF in the middle of reading a tag. Parse fails. |


A12. Flag Combination Stress
------------------------------

| #  | Test | Description |
|----|------|-------------|
| R105 | flags_64bit_8tag_progressive | Sizing=64, TagSizing=8, Operating=PROGRESSIVE. FORM with 8-byte tags, 8-byte sizes, terminated by ' END'. Parse succeeds. |
| R106 | flags_16bit_le_no_padding | Sizing=16, Typing=LE, Structuring=NO_PADDING. FORM with 2-byte LE sizes and no padding on odd chunks. Parse succeeds. |
| R107 | flags_64bit_16tag_le_unsigned_sharding | Sizing=64, TagSizing=16, Typing=LE+UNSIGNED, Structuring=SHARDING. Complex configuration. FORM with 16-byte tags, 8-byte LE sizes. Parse succeeds. |
| R108 | flags_all_defaults_explicit | ' IFF' with every flag field explicitly set to its default (0). Identical behavior to IFF-85. `iff85_locked == 0`. |


A13. Container Entity Routing
-------------------------------

| #  | Test | Description |
|----|------|-------------|
| R109 | cat_entity_delivery | FORM AAAA with CAT BBBB containing 3 FORMs. All 3 entities delivered to AAAA's `process_nested_form`. `enter_container`/`leave_container` bracket the group. |
| R110 | two_cats_group_boundaries | FORM AAAA with CAT BBBB (2 FORMs) + CAT CCCC (1 FORM). Event log shows two distinct groups with correct type tags. |
| R111 | consecutive_same_type_cats | Two consecutive CAT BBBB inside FORM AAAA. Two distinct `enter_container`/`leave_container` pairs in event log — groups are distinguishable. |
| R112 | list_cat_nested_delivery | FORM AAAA > LIST XXXX > CAT BBBB > 2 FORMs. Entities bubble through both intermediate levels to AAAA. Nesting depth tracked (LIST=1, CAT=2). |
| R113 | root_cat_entity_delivery | Root-level CAT BBBB with 2 FORMs (no parent FORM). Entities go to `session->final_entity`. No crash, no container events dispatched. |
| R114 | root_list_cat_entity_delivery | Root-level LIST > CAT > FORM (no parent FORM). Entity goes to `session->final_entity`. Multi-level root containers handled safely. |
| R115 | direct_nested_form_regression | Direct FORM-in-FORM (no CAT/LIST intermediary). Entity delivered to parent's `process_nested_form` without container events. Regression test for existing behavior. |
| R116 | list_entity_delivery | FORM AAAA with LIST BBBB containing 2 FORMs. Entities bubble to AAAA. `enter_container`/`leave_container` events fire for the LIST. |


Part B: Write Path (Generator)
================================


B1. Generator Initialization and Memory Mode
----------------------------------------------

| #  | Test | Description |
|----|------|-------------|
| W1 | create_to_data_memory_mode | CreateToData produces generator in memory mode. GetOutputData returns non-null buffer after Flush. |
| W2 | default_flags_iff85 | Newly constructed generator has IFF-85 default flags (4-byte tags, 32-bit BE, blobbed, padding on). |
| W3 | flush_empty_succeeds | Generator with no content. Flush succeeds (no open scopes or spans). Output buffer is empty. |
| W4 | flush_with_open_scope_fails | BeginForm without EndForm. Flush fails (open scope detected). |
| W5 | flush_with_open_span_fails | BeginChecksumSpan without EndChecksumSpan. Flush fails (open span detected). |


B2. WriteHeader
----------------

| #  | Test | Description |
|----|------|-------------|
| W6 | header_at_root | WriteHeader at root level. Succeeds. Output contains ' IFF' tag, size=12, version, revision, flags. |
| W7 | header_inside_container_fails | BeginForm then WriteHeader. WriteHeader fails (not at root level). |
| W8 | header_updates_generator_flags | WriteHeader with progressive flags. Generator's internal flags updated. Subsequent BeginForm uses progressive mode. |
| W9 | header_binary_layout | WriteHeader byte-for-byte: ' IFF'(4 bytes) + size(4 bytes BE, value 12) + version(2 BE) + revision(2 BE) + flags(8 BE). Compare against TestBuilder. |
| W10 | header_always_iff85_config | WriteHeader itself is always written using IFF-85 config (4-byte tags, 32-bit BE sizes), regardless of generator flags. Verify via byte layout. |


B3. Container Lifecycle — Blobbed Mode
----------------------------------------

| #  | Test | Description |
|----|------|-------------|
| W11 | form_blobbed_roundtrip | Generate IFF-85 FORM, parse result. Roundtrip succeeds. **[COVERED: test 4]** |
| W12 | form_blobbed_binary_layout | Generator output byte-identical to TestBuilder for FORM with two chunks. **[COVERED: test 22]** |
| W13 | form_empty_blobbed | BeginForm + EndForm with no chunks. Output: FORM + size(=tag_len) + type_tag. Parse succeeds. |
| W14 | list_blobbed_roundtrip | Generate LIST with PROP and FORM. Parse result succeeds. **[COVERED: test 9]** |
| W15 | cat_blobbed_roundtrip | Generate CAT with two FORMs. Parse result succeeds. **[COVERED: test 10]** |
| W16 | nested_blobbed_roundtrip | Generate LIST > LIST > FORM nesting. Parse result succeeds. **[COVERED: test 11]** |
| W17 | nested_blobbed_binary_accumulation | Three-level nesting (CAT > LIST > FORM). Generator output byte-identical to TestBuilder. Inner containers accumulated into outer accumulator. |
| W18 | blobbed_size_patching | FORM with two chunks. Verify FORM's size field equals total of type_tag + chunk1(tag+size+data+pad) + chunk2(tag+size+data+pad). |
| W19 | prop_blobbed | Generate LIST with PROP containing two property chunks, then FORM. Verify output matches TestBuilder. Parse succeeds. |


B4. Container Lifecycle — Progressive Mode
--------------------------------------------

| #  | Test | Description |
|----|------|-------------|
| W20 | form_progressive_roundtrip | WriteHeader(progressive) + BeginForm + chunks + EndForm + Flush. Parse result succeeds. `iff85_locked == 0`. **[COVERED: test 21]** |
| W21 | form_progressive_binary_layout | Progressive FORM output: FORM_tag(4) + type_tag(4) + chunks + END_tag(4) + END_size(4, value 0). Compare against TestBuilder. |
| W22 | list_progressive_roundtrip | Progressive LIST with PROP and nested FORM. All terminated by ' END'. Parse succeeds. |
| W23 | form_empty_progressive | BeginForm + EndForm in progressive mode. Output: FORM_tag + type_tag + END_tag + END_size(0). |
| W24 | nested_progressive | Progressive CAT containing two progressive FORMs. Each container terminated by its own ' END'. Parse succeeds. |


B5. Container Nesting Validation
----------------------------------

| #  | Test | Description |
|----|------|-------------|
| W25 | form_allows_chunks | WriteChunk inside FORM succeeds. |
| W26 | list_rejects_chunks | WriteChunk inside LIST fails (chunks not allowed in LIST). |
| W27 | cat_rejects_chunks | WriteChunk inside CAT fails (chunks not allowed in CAT). |
| W28 | prop_allows_chunks | WriteChunk inside PROP succeeds. |
| W29 | form_allows_nested_form | BeginForm inside FORM succeeds (FORM supports nested containers). |
| W30 | form_allows_nested_list | BeginList inside FORM succeeds (FORM supports nested containers). |
| W31 | prop_rejects_nested_container | BeginForm inside PROP fails (PROP cannot nest containers). |
| W32 | cat_rejects_prop | BeginProp inside CAT fails (PROP not allowed in CAT). |
| W33 | list_allows_all_containers | BeginForm, BeginList, BeginCat, BeginProp all succeed inside LIST. |
| W34 | cat_allows_form_list_cat | BeginForm, BeginList, BeginCat succeed inside CAT. |
| W35 | root_allows_form_list_cat | BeginForm, BeginList, BeginCat succeed at root level. BeginProp at root level fails (PROP only valid inside LIST). |
| W36 | strict_containers_match | STRICT_CONTAINERS flag set. BeginForm(ILBM) inside LIST(ILBM). Types match. Succeeds. |
| W37 | strict_containers_mismatch | STRICT_CONTAINERS flag set. BeginForm(8SVX) inside LIST(ILBM). Types don't match. Fails. |
| W38 | strict_containers_wildcard | STRICT_CONTAINERS flag set. BeginForm(ILBM) inside CAT("    "). Wildcard parent allows any child. Succeeds. |


B6. WriteChunk
---------------

| #  | Test | Description |
|----|------|-------------|
| W39 | chunk_basic | WriteChunk with 10-byte payload. Output: tag(4) + size(4, value 10) + data(10). |
| W40 | chunk_empty | WriteChunk with NULL data pointer. Output: tag(4) + size(4, value 0). No payload. |
| W41 | chunk_odd_padding | WriteChunk with 5-byte payload. Output: tag + size(5) + data(5) + pad(0x00). Parse succeeds. |
| W42 | chunk_odd_no_padding | NO_PADDING flag set. WriteChunk with 5-byte payload. Output: tag + size(5) + data(5). No pad byte. |
| W43 | chunk_at_root_fails | WriteChunk at root level (no container open). Fails. |
| W44 | chunk_binary_layout | WriteChunk output byte-identical to TestBuilder for same tag+data. |


B7. Filler and Padding
------------------------

| #  | Test | Description |
|----|------|-------------|
| W45 | filler_basic | WriteFiller(8). Output: shard_tag(4) + size(4, value 8) + zeros(8). **[COVERED: test 23]** |
| W46 | filler_zero_size | WriteFiller(0). Output: shard_tag(4) + size(4, value 0). No payload. |
| W47 | filler_binary_layout | WriteFiller output byte-identical to TestBuilder AddDirective("    ", zeros). **[COVERED: test 23]** |
| W48 | filler_odd_size_padded | WriteFiller(7). Output includes 1 pad byte after 7 zero bytes. |


B8. Shard Directive
---------------------

| #  | Test | Description |
|----|------|-------------|
| W49 | shard_basic | SHARDING enabled. WriteShard with 16-byte payload. Output: shard_tag + size(16) + data(16). |
| W50 | shard_without_flag_fails | SHARDING NOT enabled. WriteShard fails. |
| W51 | shard_roundtrip | SHARDING enabled. Generate chunk + shard. Parse with SHARDING. Decoder receives both data blocks. |


B9. VER and REV Directives
----------------------------

| #  | Test | Description |
|----|------|-------------|
| W52 | ver_directive | WriteVER with 8-byte data. Output: VER_tag(4) + size(4, value 8) + data(8). |
| W53 | rev_directive | WriteREV with 4-byte data. Output: REV_tag(4) + size(4, value 4) + data(4). |
| W54 | ver_rev_roundtrip | WriteHeader + WriteVER + WriteREV + FORM. Parse succeeds (VER/REV skipped). |


B10. DEF and REF Directives
-----------------------------

| #  | Test | Description |
|----|------|-------------|
| W55 | def_directive | WriteDEF with 4-byte identifier. Output: DEF_tag + size + payload(num_options=1, id_size=4, id_data). |
| W56 | ref_directive_single | WriteREF with 1 option, 4-byte identifier. Output: REF_tag + size + payload(num_options=1, id_size=4, id_data). |
| W57 | ref_directive_multiple | WriteREF with 3 options. Payload contains num_options=3 followed by three id entries. |
| W58 | def_ref_roundtrip | WriteHeader + WriteDEF + FORM + WriteREF. Parse succeeds (DEF and REF handled). |


B11. Checksum Span Generation
-------------------------------

| #  | Test | Description |
|----|------|-------------|
| W59 | checksum_span_blobbed | Generate FORM with CHK..SUM around a chunk in blobbed mode. Parse result. Checksum verifies. |
| W60 | checksum_span_progressive | Generate progressive FORM with CHK..SUM around a chunk. Parse result. Checksum verifies. |
| W61 | checksum_span_empty | BeginChecksumSpan immediately followed by EndChecksumSpan (no data between). SUM emitted with checksum of empty input. |
| W62 | checksum_span_nested | Two nested CHK..SUM spans. Inner span has its own checksum. Outer span covers inner CHK+data+SUM. Both verify on parse. |
| W63 | checksum_span_multiple_algorithms | BeginChecksumSpan with set of 2 algorithm IDs. SUM contains checksums for both. Parse verifies both. |
| W64 | checksum_binary_layout_chk | ' CHK' output: tag(4) + size + payload(version=1, num_ids, {id_size, id_data}...). Compare byte-for-byte against TestBuilder. |
| W65 | checksum_binary_layout_sum | ' SUM' output: tag(4) + size + payload(version=1, num_entries, {id_sz, id, sum_sz, sum}...). Compare against TestBuilder. |


B12. Encoder Framework
------------------------

### B12.1 FormEncoder

| #  | Test | Description |
|----|------|-------------|
| W66 | form_encoder_lifecycle | Register FormEncoder for ILBM. EncodeForm drives begin_encode, produce_chunk (2 chunks), produce_nested_form (done), end_encode. Output is valid FORM. **[COVERED: test 24]** |
| W67 | form_encoder_empty | FormEncoder produces 0 chunks (produce_chunk immediately sets done=1). Empty FORM emitted. |
| W68 | form_encoder_begin_fails | FormEncoder's begin_encode returns 0. EncodeForm returns 0. EndForm still called for cleanup. |
| W69 | form_encoder_produce_chunk_fails | FormEncoder's produce_chunk returns 0 on second chunk. EncodeForm returns 0. Cleanup called. |
| W70 | form_encoder_unregistered | EncodeForm called with unregistered form type. Fails (encoder not found). |

### B12.2 ChunkEncoder

| #  | Test | Description |
|----|------|-------------|
| W71 | chunk_encoder_transform | Registered ChunkEncoder for BMHD doubles each byte. EncodeForm produces chunk with transformed data. Output matches expected. **[COVERED: test 25]** |
| W72 | chunk_encoder_selective | ChunkEncoder registered for BMHD only. BODY chunk passes through untransformed. Verify BMHD transformed, BODY unchanged. |
| W73 | chunk_encoder_binary_layout | ChunkEncoder output byte-identical to TestBuilder with post-transform data. **[COVERED: test 25]** |

### B12.3 Encoder + Progressive Mode

| #  | Test | Description |
|----|------|-------------|
| W74 | encode_progressive | WriteHeader(progressive) + EncodeForm. Output includes ' END'. Parse succeeds. `iff85_locked == 0`. **[COVERED: test 26]** |


B13. Header Flag Effects on Generation
-----------------------------------------

### B13.1 Size Width

| #  | Test | Description |
|----|------|-------------|
| W75 | gen_sizing_16 | WriteHeader(sizing=16). FORM chunk sizes written as 2-byte fields. Output matches TestBuilder with 16-bit sizes. Parse succeeds. |
| W76 | gen_sizing_64 | WriteHeader(sizing=64). FORM chunk sizes written as 8-byte fields. Output matches TestBuilder. Parse succeeds. |

### B13.2 Tag Width

| #  | Test | Description |
|----|------|-------------|
| W77 | gen_tag_sizing_8 | WriteHeader(tag_sizing=8). Tags written as 8 bytes. Output matches TestBuilder with 8-byte tags. Parse succeeds. |
| W78 | gen_tag_sizing_16 | WriteHeader(tag_sizing=16). Tags written as 16 bytes. Parse succeeds. |

### B13.3 Endianness

| #  | Test | Description |
|----|------|-------------|
| W79 | gen_little_endian | WriteHeader(typing=LITTLE_ENDIAN). Size fields written in LE order. Output matches TestBuilder(is_le=1). Parse succeeds. |
| W80 | gen_le_unsigned_combined | WriteHeader(typing=LE+UNSIGNED). Both flags active. Output matches TestBuilder. Parse succeeds. |

### B13.4 Structuring

| #  | Test | Description |
|----|------|-------------|
| W81 | gen_no_padding | WriteHeader(structuring=NO_PADDING). Odd-size chunks have no pad byte. Output matches TestBuilder. Parse succeeds. |
| W82 | gen_sharding_enabled | WriteHeader(structuring=SHARDING). WriteShard succeeds. Output matches TestBuilder. |


B14. Complex Flag Combinations
---------------------------------

| #  | Test | Description |
|----|------|-------------|
| W83 | gen_64bit_8tag_progressive | Sizing=64, TagSizing=8, Operating=PROGRESSIVE. FORM with 8-byte tags, 8-byte sizes. Output parseable. |
| W84 | gen_16bit_le_no_padding | Sizing=16, Typing=LE, NO_PADDING. FORM with 2-byte LE sizes, odd chunk without padding. Output parseable. |
| W85 | gen_full_featured | Sizing=64, TagSizing=16, Typing=LE+UNSIGNED, Structuring=SHARDING. Full flag stack. Generate FORM + chunk + shard. Parse succeeds. |


B15. Round-Trip Symmetry
--------------------------

| #  | Test | Description |
|----|------|-------------|
| W86 | roundtrip_blobbed_default | Generate IFF-85 FORM, parse, verify structure matches. **[COVERED: test 4]** |
| W87 | roundtrip_progressive | Generate progressive FORM, parse, verify. **[COVERED: test 21]** |
| W88 | roundtrip_nested_containers | Generate CAT > LIST > FORM with PROP, parse, verify complete structure. |
| W89 | roundtrip_all_container_types | Generate file with FORM, LIST (with PROP), and CAT in sequence. Parse all three. |
| W90 | roundtrip_checksum | Generate CHK..SUM around chunk. Parse verifies checksum. **[Related to test 20]** |
| W91 | roundtrip_16bit_le | Generate with Sizing=16, Typing=LE. Parse the output. Verify session flags match. |
| W92 | roundtrip_64bit_8tag | Generate with Sizing=64, TagSizing=8. Parse the output. Verify. |
| W93 | roundtrip_no_padding_odd_chunk | Generate with NO_PADDING and odd-size chunk. Parse. Chunk data matches. |
| W94 | roundtrip_sharding | Generate with SHARDING enabled. Chunk + shard. Parse with decoder. Decoder receives both segments. |
| W95 | roundtrip_multiple_top_forms | Generate IFF-2025 header + two top-level FORMs. Parse both. Session completes. |
| W96 | roundtrip_def_ref | Generate header + DEF + FORM + REF(optional). Parse. DEF and REF consumed. Session completes. |
| W97 | roundtrip_ver_rev | Generate header + VER + REV + FORM. Parse. VER/REV skipped. FORM parsed. |
| W98 | roundtrip_encoder_progressive_checksum | Generate: header(progressive) + BeginChecksumSpan + EncodeForm + EndChecksumSpan + Flush. Parse verifies checksum and structure. |


B16. Bytes Written Tracking
-----------------------------

| #  | Test | Description |
|----|------|-------------|
| W99 | bytes_written_single_chunk | After WriteChunk, scope's bytes_written == tag_len + size_len + data_size + padding. |
| W100 | bytes_written_two_chunks | After two WriteChunk calls, scope's bytes_written is cumulative total. |
| W101 | bytes_written_filler | WriteFiller(8) adds tag_len + size_len + 8 to bytes_written. |
| W102 | bytes_written_nested | Nested FORM's bytes contribute to parent scope's bytes_written on EndForm. |


B17. Container Group Encoding
-------------------------------

| #  | Test | Description |
|----|------|-------------|
| W103 | roundtrip_container_groups | Generate FORM AAAA with CAT BBBB (2 FORMs) + CAT CCCC (1 FORM) via `begin_container_group`/`produce_grouped_form` FormEncoder callbacks. Parse back with ContainerAwareFormDecoder. Verify 3 entities, 7 events (ENTER BBBB, entity, entity, LEAVE BBBB, ENTER CCCC, entity, LEAVE CCCC). |


Appendix: Original Test Mapping
=================================

The original 26 tests cover 35 conformance points (9 tests verify
two aspects each). These multi-coverage mappings are marked below.

| Test # | Primary ID | Also covers | Suite                      |
|--------|------------|-------------|----------------------------|
| 1      | R1         | R28         | test_parse_basic           |
| 2      | R7         |             | test_parse_basic           |
| 3      | R20        | R52         | test_parse_basic           |
| 4      | W11        | W86         | test_generate_basic        |
| 5      | R32        | R30         | test_parse_containers      |
| 6      | R37        | R31         | test_parse_containers      |
| 7      | R35        |             | test_parse_containers      |
| 8      | R21        |             | test_parse_containers      |
| 9      | W14        |             | test_generate_containers   |
| 10     | W15        |             | test_generate_containers   |
| 11     | W16        |             | test_generate_containers   |
| 12     | R56        |             | test_parse_segments        |
| 13     | R101       |             | test_parse_segments        |
| 14     | R57        |             | test_parse_segments        |
| 15     | R74        |             | test_parse_decoders        |
| 16     | R71        |             | test_parse_decoders        |
| 17     | R40        | R77         | test_parse_decoders        |
| 18     | R61        |             | test_parse_flags           |
| 19     | R67        |             | test_parse_flags           |
| 20     | R86        |             | test_parse_checksum        |
| 21     | W20        | W87         | test_generate_advanced     |
| 22     | W12        |             | test_generate_advanced     |
| 23     | W45        | W47         | test_generate_advanced     |
| 24     | W66        |             | test_generate_encoders     |
| 25     | W71        | W73         | test_generate_encoders     |
| 26     | W74        |             | test_generate_encoders     |


Appendix: Summary Statistics
==============================

| Path       | Conformance Points | Test Functions | Multi-coverage |
|------------|--------------------|----------------|----------------|
| Read (R)   | 116                | 111            | 5 (R28,R30,R31,R52,R77) |
| Write (W)  | 103                | 99             | 4 (W47,W73,W86,W87) |
| **Total**  | **219**            | **210**        | **9** |

210 unique test functions verify 219 conformance points.
9 points share a test function with another point (multi-coverage).
