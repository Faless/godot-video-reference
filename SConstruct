#!python

import os
import SCons


def validate_godotcpp_dir(key, val, env):
    normalized = val if os.path.isabs(val) else os.path.join(env.Dir("#").abspath, val)
    if not os.path.isdir(normalized):
        raise UserError("GDExtension directory ('%s') does not exist: %s" % (key, val))

env = Environment()
opts = Variables(["customs.py"], ARGUMENTS)

# Define our options
opts.Add(BoolVariable("test", "Build to test dir", "yes"))
opts.Add(PathVariable("target_path", "The path where the lib is installed.", "bin/", PathVariable.PathAccept))
opts.Add(PathVariable("test_path", "The path where the test is installed.", "test/addons/godot_video_reference/bin/", PathVariable.PathAccept))
opts.Add(PathVariable("target_name", "The library name.", "libgdvideo", PathVariable.PathAccept))
opts.Add(BoolVariable("vsproj", "Generate a project for Visual Studio", "no"))
opts.Add(BoolVariable("builtin_opus", "Use Built-in Opus", "yes"))
opts.Add(BoolVariable("builtin_libogg", "Use Built-in LibOgg", "yes"))
opts.Add(
    PathVariable(
        "godot_cpp",
        "Path to the directory containing Godot CPP folder",
        None,
        validate_godotcpp_dir,
    )
)
opts.Update(env)
Help(opts.GenerateHelpText(env))

# Import godot_cpp env and use it as our base environment
sconstruct = env.get("godot_cpp", "godot-cpp") + "/SConstruct"
cpp_env = SConscript(sconstruct)
env = cpp_env.Clone()
# Re-apply our options
opts.Update(env)

# Local dependency paths, adapt them to your setup
godot_headers_path = "godot-cpp/gdextension/"
cpp_bindings_path = "godot-cpp/"
cpp_library = "libgodot-cpp"

# For the reference:
# - CCFLAGS are compilation flags shared between C and C++
# - CFLAGS are for C-specific compilation flags
# - CXXFLAGS are for C++-specific compilation flags
# - CPPFLAGS are for pre-processor flags
# - CPPDEFINES are for pre-processor defines
# - LINKFLAGS are for linking flags

# tweak this if you want to use different folders, or more folders, to store your source code in.
env.Append(CPPPATH=["src/"])

Export('env')

def glob_filenames(pattern):
    from glob import glob
    return glob(pattern)

env.__class__.sources = []
env.__class__.modules_sources = []
env.__class__.includes = []

sources = glob_filenames("src/*.cpp")
includes = glob_filenames("src/*.h")

env.sources += sources
env.includes = includes

def add_source_files(self, sources, regex):
    files = []
    if type(regex) == list:
        files += regex
    elif type(regex) == str:
        files += glob_filenames(regex)

    # Add each path as compiled Object following environment (self) configuration
    for path in files:
        obj = self.SharedObject(path)
        if obj in sources:
            print('WARNING: Shared Object "{}" already included in environment sources.'.format(obj))
            continue
        sources.append(obj)


env.__class__.add_source_files = add_source_files 

SConscript("thirdparty/SCsub")

env.Prepend(CPPPATH=["thirdparty/"])
env.Prepend(CPPPATH=["thirdparty/libsimplewebm"])
env.Prepend(CPPPATH=["thirdparty/libsimplewebm/libwebm"])
env.Prepend(CPPPATH=["thirdparty/libvpx"])

target_name = ""
if env["target"] in ("editor", "template_debug"):
    lib_target = env["target_path"]
    target_name = "Debug"
else:
    lib_target = env["target_path"]
    target_name = "Release"

if env["test"]:
    lib_target = env["test_path"]

result_name = "{}{}{}{}".format(lib_target, env["target_name"], env["suffix"], env["SHLIBSUFFIX"])
library = env.SharedLibrary(target = result_name, source=env.sources+env.modules_sources)

Default(library)

# print (env.modules_sources)

if env["vsproj"]:
    vsproj = env.MSVSProject(target = 'godot_video_reference' + env['MSVSPROJECTSUFFIX'],
        srcs = env.sources + env.modules_sources,
        incs = env.includes,
        localincs = [],
        resources = [],
        misc = ['LICENSE','README.md','.clang-format','.gitignore','.gitmodules'],
        buildtarget = library,
        variant = [target_name + '|'+env['msvc_arch']] * len(library),
    )
