#ld script to make i386 Linux kernel

# ld script to make i386 Linux kernel #
```

OUTPUT_FORMAT("elf32-i386", "elf32-i386", "elf32-i386")
OUTPUT_ARCH(i386)
ENTRY(phys_startup_32)
jiffies = jiffies_64;
PHDRS {
 text PT_LOAD FLAGS(5); /* R_E */
 data PT_LOAD FLAGS(7); /* RWE */
 note PT_NOTE FLAGS(0); /* ___ */
}
SECTIONS
{
  . = 0xC0000000 + ((0x100000 + (0x100000 - 1)) & ~(0x100000 - 1));
  phys_startup_32 = startup_32 - 0xC0000000;
  .text.head : AT(ADDR(.text.head) - 0xC0000000) {
   _text = .; /* Text and read-only data */
 *(.text.head)
  } :text = 0x9090
  /* read-only */
  .text : AT(ADDR(.text) - 0xC0000000) {
 . = ALIGN((1 << 12)); /* not really needed, already page aligned */
 *(.text.page_aligned)
 . = ALIGN(8); *(.text) *(.ref.text) *(.text.init.refok) *(.exit.text.refok) *(.devinit.text) *(.devexit.text) *(.cpuinit.text) *(.cpuexit.text)
 . = ALIGN(8); __sched_text_start = .; *(.sched.text) __sched_text_end = .;
 . = ALIGN(8); __lock_text_start = .; *(.spinlock.text) __lock_text_end = .;
 . = ALIGN(8); __kprobes_text_start = .; *(.kprobes.text) __kprobes_text_end = .;
 *(.fixup)
 *(.gnu.warning)
   _etext = .; /* End of text section */
  } :text = 0x9090
  . = ALIGN(16); /* Exception table */
  __ex_table : AT(ADDR(__ex_table) - 0xC0000000) {
   __start___ex_table = .;
  *(__ex_table)
   __stop___ex_table = .;
  }
  .notes : AT(ADDR(.notes) - 0xC0000000) { __start_notes = .; *(.note.*) __stop_notes = .; } :text :note
  . = ALIGN(8); __bug_table : AT(ADDR(__bug_table) - 0xC0000000) { __start___bug_table = .; *(__bug_table) __stop___bug_table = .; } :text
  . = ALIGN(4);
  .tracedata : AT(ADDR(.tracedata) - 0xC0000000) {
   __tracedata_start = .;
 *(.tracedata)
   __tracedata_end = .;
  }
  . = ALIGN((4096)); .rodata : AT(ADDR(.rodata) - 0xC0000000) { __start_rodata = .; *(.rodata) *(.rodata.*) *(__vermagic) *(__markers_strings) } .rodata1 : AT(ADDR(.rodata1) - 0xC0000000) { *(.rodata1) } .pci_fixup : AT(ADDR(.pci_fixup) - 0xC0000000) { __start_pci_fixups_early = .; *(.pci_fixup_early) __end_pci_fixups_early = .; __start_pci_fixups_header = .; *(.pci_fixup_header) __end_pci_fixups_header = .; __start_pci_fixups_final = .; *(.pci_fixup_final) __end_pci_fixups_final = .; __start_pci_fixups_enable = .; *(.pci_fixup_enable) __end_pci_fixups_enable = .; __start_pci_fixups_resume = .; *(.pci_fixup_resume) __end_pci_fixups_resume = .; } .rio_route : AT(ADDR(.rio_route) - 0xC0000000) { __start_rio_route_ops = .; *(.rio_route_ops) __end_rio_route_ops = .; } __ksymtab : AT(ADDR(__ksymtab) - 0xC0000000) { __start___ksymtab = .; *(__ksymtab) __stop___ksymtab = .; } __ksymtab_gpl : AT(ADDR(__ksymtab_gpl) - 0xC0000000) { __start___ksymtab_gpl = .; *(__ksymtab_gpl) __stop___ksymtab_gpl = .; } __ksymtab_unused : AT(ADDR(__ksymtab_unused) - 0xC0000000) { __start___ksymtab_unused = .; *(__ksymtab_unused) __stop___ksymtab_unused = .; } __ksymtab_unused_gpl : AT(ADDR(__ksymtab_unused_gpl) - 0xC0000000) { __start___ksymtab_unused_gpl = .; *(__ksymtab_unused_gpl) __stop___ksymtab_unused_gpl = .; } __ksymtab_gpl_future : AT(ADDR(__ksymtab_gpl_future) - 0xC0000000) { __start___ksymtab_gpl_future = .; *(__ksymtab_gpl_future) __stop___ksymtab_gpl_future = .; } __kcrctab : AT(ADDR(__kcrctab) - 0xC0000000) { __start___kcrctab = .; *(__kcrctab) __stop___kcrctab = .; } __kcrctab_gpl : AT(ADDR(__kcrctab_gpl) - 0xC0000000) { __start___kcrctab_gpl = .; *(__kcrctab_gpl) __stop___kcrctab_gpl = .; } __kcrctab_unused : AT(ADDR(__kcrctab_unused) - 0xC0000000) { __start___kcrctab_unused = .; *(__kcrctab_unused) __stop___kcrctab_unused = .; } __kcrctab_unused_gpl : AT(ADDR(__kcrctab_unused_gpl) - 0xC0000000) { __start___kcrctab_unused_gpl = .; *(__kcrctab_unused_gpl) __stop___kcrctab_unused_gpl = .; } __kcrctab_gpl_future : AT(ADDR(__kcrctab_gpl_future) - 0xC0000000) { __start___kcrctab_gpl_future = .; *(__kcrctab_gpl_future) __stop___kcrctab_gpl_future = .; } __ksymtab_strings : AT(ADDR(__ksymtab_strings) - 0xC0000000) { *(__ksymtab_strings) } __init_rodata : AT(ADDR(__init_rodata) - 0xC0000000) { *(.ref.rodata) *(.devinit.rodata) *(.devexit.rodata) *(.cpuinit.rodata) *(.cpuexit.rodata) } __param : AT(ADDR(__param) - 0xC0000000) { __start___param = .; *(__param) __stop___param = .; . = ALIGN((4096)); __end_rodata = .; } . = ALIGN((4096));
  /* writeable */
  . = ALIGN((1 << 12));
  .data : AT(ADDR(.data) - 0xC0000000) { /* Data */
 *(.data) *(.data.init.refok) *(.ref.data) *(.devinit.data) *(.devexit.data) *(.cpuinit.data) *(.cpuexit.data) . = ALIGN(8); __start___markers = .; *(__markers) __stop___markers = .;
 CONSTRUCTORS
 } :data
  . = ALIGN((1 << 12));
  .data_nosave : AT(ADDR(.data_nosave) - 0xC0000000) {
   __nosave_begin = .;
 *(.data.nosave)
   . = ALIGN((1 << 12));
   __nosave_end = .;
  }
  . = ALIGN((1 << 12));
  .data.page_aligned : AT(ADDR(.data.page_aligned) - 0xC0000000) {
 *(.data.page_aligned)
 *(.data.idt)
  }
  . = ALIGN(32);
  .data.cacheline_aligned : AT(ADDR(.data.cacheline_aligned) - 0xC0000000) {
 *(.data.cacheline_aligned)
  }
  /* rarely changed data like cpu maps */
  . = ALIGN(32);
  .data.read_mostly : AT(ADDR(.data.read_mostly) - 0xC0000000) {
 *(.data.read_mostly)
 _edata = .; /* End of data section */
  }
  . = ALIGN((8192)); /* init_task */
  .data.init_task : AT(ADDR(.data.init_task) - 0xC0000000) {
 *(.data.init_task)
  }
  /* might get freed after init */
  . = ALIGN((1 << 12));
  .smp_locks : AT(ADDR(.smp_locks) - 0xC0000000) {
   __smp_locks = .;
 *(.smp_locks)
 __smp_locks_end = .;
  }
  /* will be freed after init
   * Following ALIGN() is required to make sure no other data falls on the
   * same page where __smp_alt_end is pointing as that page might be freed
   * after boot. Always make sure that ALIGN() directive is present after
   * the section which contains __smp_alt_end.
   */
  . = ALIGN((1 << 12));
  /* will be freed after init */
  . = ALIGN((1 << 12)); /* Init code and data */
  .init.text : AT(ADDR(.init.text) - 0xC0000000) {
   __init_begin = .;
 _sinittext = .;
 *(.init.text) *(.meminit.text)
 _einittext = .;
  }
  .init.data : AT(ADDR(.init.data) - 0xC0000000) {
 *(.init.data) *(.meminit.data) *(.meminit.rodata)
  }
  . = ALIGN(16);
  .init.setup : AT(ADDR(.init.setup) - 0xC0000000) {
   __setup_start = .;
 *(.init.setup)
   __setup_end = .;
   }
  .initcall.init : AT(ADDR(.initcall.init) - 0xC0000000) {
   __initcall_start = .;
 *(.initcall0.init) *(.initcall0s.init) *(.initcall1.init) *(.initcall1s.init) *(.initcall2.init) *(.initcall2s.init) *(.initcall3.init) *(.initcall3s.init) *(.initcall4.init) *(.initcall4s.init) *(.initcall5.init) *(.initcall5s.init) *(.initcallrootfs.init) *(.initcall6.init) *(.initcall6s.init) *(.initcall7.init) *(.initcall7s.init)
   __initcall_end = .;
  }
  .con_initcall.init : AT(ADDR(.con_initcall.init) - 0xC0000000) {
   __con_initcall_start = .;
 *(.con_initcall.init)
   __con_initcall_end = .;
  }
  .x86cpuvendor.init : AT(ADDR(.x86cpuvendor.init) - 0xC0000000) {
 __x86cpuvendor_start = .;
 *(.x86cpuvendor.init)
 __x86cpuvendor_end = .;
  }
  .security_initcall.init : AT(ADDR(.security_initcall.init) - 0xC0000000) { __security_initcall_start = .; *(.security_initcall.init) __security_initcall_end = .; }
  . = ALIGN(4);
  .altinstructions : AT(ADDR(.altinstructions) - 0xC0000000) {
   __alt_instructions = .;
 *(.altinstructions)
 __alt_instructions_end = .;
  }
  .altinstr_replacement : AT(ADDR(.altinstr_replacement) - 0xC0000000) {
 *(.altinstr_replacement)
  }
  . = ALIGN(4);
  .parainstructions : AT(ADDR(.parainstructions) - 0xC0000000) {
   __parainstructions = .;
 *(.parainstructions)
   __parainstructions_end = .;
  }
  /* .exit.text is discard at runtime, not link time, to deal with references
     from .altinstructions and .eh_frame */
  .exit.text : AT(ADDR(.exit.text) - 0xC0000000) {
 *(.exit.text) *(.memexit.text)
  }
  .exit.data : AT(ADDR(.exit.data) - 0xC0000000) {
 *(.exit.data) *(.memexit.data) *(.memexit.rodata)
  }
  . = ALIGN((1 << 12));
  .init.ramfs : AT(ADDR(.init.ramfs) - 0xC0000000) {
 __initramfs_start = .;
 *(.init.ramfs)
 __initramfs_end = .;
  }
  . = ALIGN((1 << 12));
  .data.percpu : AT(ADDR(.data.percpu) - 0xC0000000) {
 __per_cpu_start = .;
 *(.data.percpu)
 *(.data.percpu.shared_aligned)
 __per_cpu_end = .;
  }
  . = ALIGN((1 << 12));
  /* freed after init ends here */
  .bss : AT(ADDR(.bss) - 0xC0000000) {
 __init_end = .;
 __bss_start = .; /* BSS */
 *(.bss.page_aligned)
 *(.bss)
 . = ALIGN(4);
 __bss_stop = .;
   _end = . ;
 /* This is where the kernel creates the early boot page tables */
 . = ALIGN((1 << 12));
 pg0 = . ;
  }
  /* Sections to be discarded */
  /DISCARD/ : {
 *(.exitcall.exit)
 }
  .stab 0 : { *(.stab) } .stabstr 0 : { *(.stabstr) } .stab.excl 0 : { *(.stab.excl) } .stab.exclstr 0 : { *(.stab.exclstr) } .stab.index 0 : { *(.stab.index) } .stab.indexstr 0 : { *(.stab.indexstr) } .comment 0 : { *(.comment) }
  .debug 0 : { *(.debug) } .line 0 : { *(.line) } .debug_srcinfo 0 : { *(.debug_srcinfo) } .debug_sfnames 0 : { *(.debug_sfnames) } .debug_aranges 0 : { *(.debug_aranges) } .debug_pubnames 0 : { *(.debug_pubnames) } .debug_info 0 : { *(.debug_info .gnu.linkonce.wi.*) } .debug_abbrev 0 : { *(.debug_abbrev) } .debug_line 0 : { *(.debug_line) } .debug_frame 0 : { *(.debug_frame) } .debug_str 0 : { *(.debug_str) } .debug_loc 0 : { *(.debug_loc) } .debug_macinfo 0 : { *(.debug_macinfo) } .debug_weaknames 0 : { *(.debug_weaknames) } .debug_funcnames 0 : { *(.debug_funcnames) } .debug_typenames 0 : { *(.debug_typenames) } .debug_varnames 0 : { *(.debug_varnames) }
}
```