import os

opts = Variables( 'build/scons/SConfig', ARGUMENTS )

opts.AddVariables(
	EnumVariable( 'config',         'Build configuration', 'debug', allowed_values=( 'debug', 'release', 'profile', 'deploy' ) ),
	EnumVariable( 'arch',           'Architecture to compile for', '', allowed_values=( '', 'x86', 'x86_64' ) ),
	EnumVariable( 'sse',            'SSE instruction set to use', '2', allowed_values=( '2', '3', '4' ) ),
	EnumVariable( 'platform',       'Platform to compile for', '', allowed_values=( '', 'linux', 'win32', 'win64', 'macosx', 'raspberrypi' ) ),
	( 'libsuffix',                  'Extra library name suffix', '' ),
	EnumVariable( 'tools',          'Tools to use to build', 'gnu', allowed_values=( 'intel', 'gnu', 'msvc', 'clang' ) ),
	( 'foundationpath',             'Path to foundation library', '' ),
)

baseenv = Environment( variables=opts )

#for item in sorted(baseenv.Dictionary().items()):
#	print "construction variable = '%s', value = '%s'" % item

#print "HOST_ARCH: %s" % baseenv['HOST_ARCH']
#print "TARGET_ARCH: %s" % baseenv['TARGET_ARCH']
#print "PLATFORM: %s" % baseenv['PLATFORM']

if ( "%s" % baseenv['HOST_ARCH'] ) == 'None':
	if baseenv['PLATFORM'] == 'win32':
		baseenv['HOST_ARCH'] = 'x86'
	else:
		baseenv['HOST_ARCH'] = os.popen("arch").read().rstrip()

if baseenv['PLATFORM'] == 'win32':
	if baseenv['tools'] == 'gnu':
		baseenv['toolslist'] = ['mingw']
	if baseenv['tools'] == 'intel':
		baseenv['toolslist'] = ['default','intelc','mslib']
elif baseenv['PLATFORM'] == 'posix':
	if baseenv['tools'] == 'gnu':
		baseenv['toolslist'] = ['default']
	else:
		baseenv['toolslist'] = ['default']
	if baseenv['arch'] == 'arm':
		baseenv['TARGET_ARCH'] = 'arm'
	elif baseenv['arch'] == 'x86_64':
	   	baseenv['TARGET_ARCH'] = 'x86_64'
	elif baseenv['arch'] == 'x86':
		baseenv['TARGET_ARCH'] = 'x86'
	else:
		baseenv['TARGET_ARCH'] = baseenv['HOST_ARCH']
else:
	baseenv['toolslist'] = ['default']

if baseenv['platform'] == 'win64':
	baseenv['TARGET_ARCH'] = 'x86_64'

env = Environment(
	variables=opts,
	CPPPATH=['#'],
	TARGET_ARCH=baseenv['TARGET_ARCH'],
	HOST_ARCH=baseenv['HOST_ARCH'],
	tools=baseenv['toolslist']
)

#for item in sorted(baseenv.Dictionary().items()):
#	print "construction variable = '%s', value = '%s'" % item

env['platformsuffix'] = ''

if env['PLATFORM'] == 'posix':
	if env['platform'] == '':
		env['platform'] = 'linux'
	env['arch'] = env['TARGET_ARCH']
	env['ENV']['TERM'] = os.environ['TERM']
	#if env['platform'] == 'raspberrypi':
	#	env['CC'] = 'clang'
	#	env['CXX'] = 'clang++'
	#	env['LD'] = 'llvm-ld'
if env['TARGET_ARCH'] == 'x86' and env['arch'] != 'x86_64':
	env['arch'] = 'x86'
if env['TARGET_ARCH'] == 'i686' and env['arch'] != 'x86_64':
	env['arch'] = 'x86'
if env['TARGET_ARCH'] == 'x86_64':
	env['arch'] = 'x86_64'
if env['TARGET_OS'] == 'win32' or env['PLATFORM'] == 'win32':
	if env['arch'] == 'x86_64':
		env['platform'] = 'win64'
	elif env['arch'] == 'x86':
		env['platform'] = 'win32'
	else:
		if env['HOST_ARCH'] == 'x86_64':
			env['platform'] = 'win64'
			env['arch'] = 'x86_64'
		else:
			env['platform'] = 'win32'
			env['arch'] = 'x86'
if env['platform'] == 'linux':
	if env['arch'] == 'x86':
		env['platformsuffix'] = '32'
	elif env['arch'] == 'x86_64':
		env['platformsuffix'] = '64'
	else:
		env['platformsuffix'] = env['arch']

env['buildprofile'] = env['config']

#print "HOST_ARCH: %s" % baseenv['HOST_ARCH']
#print "TARGET_ARCH: %s" % baseenv['TARGET_ARCH']
#print "PLATFORM: %s" % baseenv['PLATFORM']

print "Building on " + env['PLATFORM'] + " (" + env['HOST_ARCH'] + ") for " + env['platform'] + " (" + env['arch'] + ") using " + env['tools']

Help( opts.GenerateHelpText( env ) )

# SETUP DEFAULT COMPILER AND LINKER FLAGS SHARED BY ALL CONFIGS
if env['CC'] == 'gcc' or env['CC'] == 'clang':
	env.Append( CFLAGS=['-std=gnu99','-W','-Wall','-Wcast-align','-Wcast-qual','-Wchar-subscripts','-Winline','-Wpointer-arith','-Wredundant-decls','-Wshadow','-Wwrite-strings','-Wno-variadic-macros','-Wno-long-long','-Wno-format','-Wno-unused','-Wundef','-Wstrict-aliasing','-Wno-missing-field-initializers','-Wno-missing-braces','-Wno-unused-parameter','-ftabstop=4','-fstrict-aliasing'] )
	if env['platform'] == 'raspberrypi':
		env.Append( CFLAGS=['-pedantic','-Werror'] )
		env.Append( CFLAGS=['-march=armv6j','-mfloat-abi=hard','-mfpu=vfp','-mtune=arm1176jzf-s'] )
		env.Append( CXXFLAGS=['-march=armv6j','-mfloat-abi=hard','-mfpu=vfp','-mtune=arm1176jzf-s'] )
		env.Append( CPPPATH=['/opt/vc/include','/opt/vc/include/interface/vcos/pthreads'] )
		env.Append( LIBPATH=['/opt/vc/lib'] )
		#echo 'SUBSYSTEM=="vchiq",GROUP="video",MODE="0660"' > /etc/udev/rules.d/10-vchiq-permissions.rules
	    #usermod -a -G video [your_username]
	if env['arch'] == 'x86':
		env.Append( CCFLAGS=['-m32'] )
		env.Append( LINKFLAGS=['-m32'] )
		env.Append( CCFLAGS=['-msse' + env['sse']] )
	if env['arch'] == 'x86_64':
		env.Append( CCFLAGS=['-m64'] )
		env.Append( LINKFLAGS=['-m64'] )
		env.Append( CCFLAGS=['-msse' + env['sse']] )
	env.Append( LINKFLAGS=['-pthread'] )

if env['CC'] == 'icl':
	env.Append( CFLAGS=['/Zi','/W3','/WX','/Oi','/Quse-intel-optimized-headers','/MT','/GS-','/fp:fast=2','/QxSSE3','/GR-','/Qstd=c99','/Qrestrict','/Qansi-alias'] )

if env['CC'] == 'cl':
	env.Append( LINKFLAGS=[ '/MACHINE:X86' ] )


# SETUP DEFAULT ENVIRONMENT
#if env['PLATFORM'] == 'win32':
#	# AVOID CMDLINE LENGTH OVERFLOW
#	import win32file 
#	import win32event  
#	import win32process 
#	import win32security 
#	import string 
#	import shutil
#
#	def my_spawn(sh, escape, cmd, args, spawnenv): 
#		for var in spawnenv:  
#			spawnenv[var] = spawnenv[var].encode('ascii', 'replace') 
#		sAttrs = win32security.SECURITY_ATTRIBUTES() 
#		StartupInfo = win32process.STARTUPINFO() 
#		newargs = string.join( args[1:], ' ' ) #map(escape, args[1:]), ' ') 
#		cmdline = cmd + " " + newargs 
#		exit_code = 0
#		# check for any special operating system commands 
#		if cmd == 'del':
#			for arg in args[1:]: 
#				win32file.DeleteFile(arg) 
#		elif cmd == 'copy':
#			shutil.copyfile(args[1].strip('"'),args[2].strip('"'))
#		else:
#			# otherwise execute the command.
#			hProcess, hThread, dwPid, dwTid = win32process.CreateProcess(None, cmdline, None, None, 1, 0, spawnenv, None, StartupInfo) 
#			win32event.WaitForSingleObject(hProcess, win32event.INFINITE) 
#			exit_code = win32process.GetExitCodeProcess(hProcess)
#			win32file.CloseHandle(hProcess); 
#			win32file.CloseHandle(hThread); 
#		return exit_code  
#	env['SPAWN'] = my_spawn

if env['CC'] == 'gcc':
	env['BUILDERS']['PCH'] = Builder( action = '$CXX -x c++-header $CXXFLAGS $_CPPINCFLAGS $_CPPDEFFLAGS -o $TARGET $SOURCE', suffix = '.h.gch', src_suffix = '.h' )


# SETUP DEBUG ENVRIONMENT
if env['buildprofile'] == 'debug':
	print "Building DEBUG configuration"
	env.Append( CPPDEFINES=['BUILD_DEBUG=1'] )
	env['buildpath'] = 'debug'
	if env['CC'] == 'gcc' or env['CC'] == 'clang':
		env.Append( CFLAGS=['-g','-fno-math-errno','-ffinite-math-only'] )
		if env['CC'] == 'gcc':
			env.Append( CFLAGS=['-funsafe-math-optimizations','-fno-trapping-math'] )
	if env['CC'] == 'icl':
		env.Append( CFLAGS=['/Od'] )

# SETUP RELEASE ENVIRONMENT
elif env['buildprofile'] == 'release':
	print "Building RELEASE configuration"
	env.Append( CPPDEFINES=['BUILD_RELEASE=1'] )
	env['buildpath'] = 'release';
	if env['CC'] == 'gcc' or env['CC'] == 'clang':
		env.Append( CFLAGS=['-g','-O3','-ffast-math','-funit-at-a-time','-fno-math-errno','-funsafe-math-optimizations','-ffinite-math-only','-fno-trapping-math','-funroll-loops'] )
	if env['CC'] == 'icl':
		env.Append( CFLAGS=['/O3','/Ob2','/Ot','/GT','/GF'] )

# SETUP PROFILE ENVIRONMENT
elif env['buildprofile'] == 'profile':
	print "Building PROFILE configuration"
	env.Append( CPPDEFINES=['BUILD_PROFILE=1'] )
	env['buildpath'] = 'profile';
	if env['CC'] == 'gcc' or env['CC'] == 'clang':
		env.Append( CFLAGS=['-g','-O6','-ffast-math','-funit-at-a-time','-fno-math-errno','-funsafe-math-optimizations','-ffinite-math-only','-fno-trapping-math','-funroll-loops'] )
	if env['CC'] == 'icl':
		env.Append( CFLAGS=['/O3','/Ob2','/Ot','/GT','/GF'] )

# SETUP DEPLOY ENVIRONMENT
elif env['buildprofile'] == 'deploy':
	print "Building DEPLOY configuration"
	env.Append( CPPDEFINES=['BUILD_DEPLOY=1'] )
	env['buildpath'] = 'deploy';
	if env['CC'] == 'gcc' or env['CC'] == 'clang':
		env.Append( CFLAGS=['-O6','-ffast-math','-funit-at-a-time','-fno-math-errno','-funsafe-math-optimizations','-ffinite-math-only','-fno-trapping-math','-funroll-loops'] )
	if env['CC'] == 'icl':
		env.Append( CFLAGS=['/O3','/Ob2','/Ot','/GT','/GF'] )


# SETUP COMMON ENVIRONMENT
if env['platform'] == 'raspberrypi':
	env.Append( CPPDEFINES=['__raspberrypi__=1'] )

if not env['foundationpath'] == '':
	env.Append( CPPPATH=[env['foundationpath']] )
	env.Append( LIBPATH=[env['foundationpath'] + '/lib/${platform}${platformsuffix}/${buildprofile}'] )

env['buildpath'] = env['buildpath'] + '-' + env['platform'] + env['platformsuffix']

Export("env")

VariantDir( 'build/scons/%s/game' % env['buildpath'] , 'game', duplicate=0 )
SConscript( 'build/scons/%s/game/SConscript' % env['buildpath']  )

VariantDir( 'build/scons/%s/template' % env['buildpath'] , 'template', duplicate=0 )
SConscript( 'build/scons/%s/template/SConscript' % env['buildpath']  )

#if env['buildprofile'] != 'profile' and env['buildprofile'] != 'deploy':

#   VariantDir( 'build/scons/%s/tools' % env['buildpath'] , 'tools', duplicate=0 )
#   SConscript( 'build/scons/%s/tools/SConscript' % env['buildpath']  )
