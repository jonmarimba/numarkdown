
(set DEVROOT 
     (ifDarwin (then (NSString stringWithShellCommand:"xcode-select -print-path"))
               (else nil)))

;; source files
(set @c_files     (filelist "^objc/.*\.c$"))
(set @m_files     (filelist "^objc/.*.m$"))
(set @nu_files 	  (filelist "^nu/.*nu$"))
(set @frameworks  '("Cocoa" "Nu"))
(set @inc_dirs   (NSMutableArray arrayWithList:(list "/usr/include")))
(set @lib_dirs (array)) ;;  (NSMutableArray arrayWithList:(list "/usr/lib")))

;; framework description
(set @framework 			 "NuMarkdown")
(set @framework_identifier   "nu.programming.markdown")
(set @framework_creator_code "????")
(set @framework_initializer  "MarkdownInit")

;; build configuration
(set @cc "gcc")
(set @cc "#{DEVROOT}/usr/bin/clang")

(set @sdkflags "")
(set @sdk
     (cond ((NSFileManager directoryExistsNamed:"#{DEVROOT}/SDKs/MacOSX10.7.sdk")
            (set @sdkflags "-D__OBJC2__ -DSNOWLEOPARD")
            ("-isysroot #{DEVROOT}/SDKs/MacOSX10.7.sdk"))
           ((NSFileManager directoryExistsNamed:"#{DEVROOT}/SDKs/MacOSX10.6.sdk")
            (set @sdkflags "-D__OBJC2__ -DSNOWLEOPARD")
            ("-isysroot #{DEVROOT}/SDKs/MacOSX10.6.sdk"))
           ((NSFileManager directoryExistsNamed:"#{DEVROOT}/SDKs/MacOSX10.5.sdk")
            (set @sdkflags "-D__OBJC2__")
            ("-isysroot #{DEVROOT}/SDKs/MacOSX10.5.sdk"))
           ((NSFileManager directoryExistsNamed:"#{DEVROOT}/SDKs/MacOSX10.4u.sdk")
            ("-isysroot #{DEVROOT}/SDKs/MacOSX10.4u.sdk"))
           (else "")))

(set @cflags "-Wall -g -fPIC")

(ifDarwin
         (then (set @cflags (+ @cflags " -g -O2 -DMACOSX #{@sdk} #{@sdkflags}"))
               (set @mflags_nogc "-fobjc-exceptions")
               (set @mflags (+ @mflags_nogc " -fobjc-gc"))) ;; To use garbage collection, add this flag: "-fobjc-gc"
         (else (set @cflags "-Wall -g -std=gnu99 -fPIC")
               (set @mflags ((NSString stringWithShellCommand:"gnustep-config --objc-flags") chomp))))

(ifLinux       (then (set @cflags (+ @cflags " -DLINUX"))))
(ifFreeBSD     (then (set @cflags (+ @cflags " -DFREEBSD"))))
(ifOpenSolaris (then (set @cflags (+ @cflags " -DOPENSOLARIS"))))

(ifDarwin
         (then (set @arch '()))) ;; optionally add "ppc" or "ppc64" to the list

(if (or isSnowLeopard isLion)
	(then (set @arch (append @arch '("x86_64")))))

(set @includes
     ((@inc_dirs map: (do (inc) " -I#{inc}")) join))
(set @ldflags
     ((list
           ((@frameworks map: (do (framework) " -framework #{framework}")) join)
           ((@libs map: (do (lib) " -l#{lib}")) join)
           (ifDarwin
                    (then ((@lib_dirs map:
                        (do (libdir) " -L#{libdir}")) join))
                    (else
                        (if (isOpenSolaris)
                          (then
                            ((@lib_dirs map:
                                 (do (libdir) " -L#{libdir} ")) join))
                          (else 
                            ((@lib_dirs map:
                                 (do (libdir) " -L#{libdir} -Wl,--rpath #{libdir}")) join))))))
     join))


(compilation-tasks)
(framework-tasks)

(task "bin/nudown" is
      (SH "gcc objc/nudown.m -o bin/nudown -framework Cocoa -framework Nu"))

(task "test" => "framework" is
      (SH "nutest test/test_markdown.nu"))

(task "clean" is
      (SH "rm -f test/SimpleTests/*.html")
      (SH "rm -f test/MarkdownTests/*.html"))

(task "clobber" => "clean" is
      (system "rm -rf #{@framework_dir}"))

(task "default" => "framework")

(task "install" => "framework" is
      (SH "sudo cp bin/nudown /usr/local/bin/nudown")
      (SH "ditto #{@framework_dir} /Library/Frameworks/#{@framework_dir}"))