PROJECT(iodoom3)
CMAKE_MINIMUM_REQUIRED(VERSION 2.6.4)

INCLUDE(CheckCCompilerFlag)

IF(WIN32 AND NOT WINDOWS)
    SET(WINDOWS TRUE)
ENDIF(WIN32 AND NOT WINDOWS)
IF(APPLE AND NOT MACOSX)
    SET(MACOSX TRUE)
ENDIF(APPLE AND NOT MACOSX)
IF(CMAKE_SYSTEM_NAME STREQUAL "Linux")
    SET(LINUX TRUE)
ENDIF(CMAKE_SYSTEM_NAME STREQUAL "Linux")

IF(NOT CMAKE_BUILD_TYPE)
    SET(IODOOM3_DEBUG_BUILD ON)
ENDIF(NOT CMAKE_BUILD_TYPE)
IF(CMAKE_BUILD_TYPE STREQUAL "Debug")
    SET(IODOOM3_DEBUG_BUILD ON)
ENDIF(CMAKE_BUILD_TYPE STREQUAL "Debug")

# See what CPU we appear to be targeting...
#  It would be nice if CMake handled this for us.
MACRO(CHECK_CPU _CDEFS _NAME _VALUE)
    CHECK_C_SOURCE_COMPILES("
        #if ( ${_CDEFS} )
        int main(int argc, char **argv) { int yup = 1; return 0; }
        #else
        #error Not targeting this CPU architecture.
        #endif
        " ${_VALUE}
    )

    IF(${_VALUE})
        IF(IODOOM3_CHOSE_CPU)
            MESSAGE(STATUS "We appear to see two different CPU architectures!")
            MESSAGE(STATUS "We saw '${IODOOM3_CPU}' and '${_NAME}'.")
            MESSAGE(FATAL_ERROR "Please fix this before continuing.")
        ENDIF(IODOOM3_CHOSE_CPU)
        SET(IODOOM3_CHOSE_CPU TRUE)
        SET(IODOOM3_CPU ${_NAME})
    ENDIF(${_VALUE})
ENDMACRO(CHECK_CPU)

SET(IODOOM3_X86 TRUE) # !!! FIXME
SET(IODOOM3_CHOSE_CPU TRUE)
SET(IODOOM3_CPU "x86")
#CHECK_CPU(
#    "defined(__i386__) || defined(__i686__) || defined(_M_X86) || defined(i386)"
#    "x86" IODOOM3_X86
#)
#
#CHECK_CPU(
#    "defined(__x86_64__) || defined(_M_X64)"
#    "amd64" IODOOM3_AMD64
#)

IF(NOT IODOOM3_CHOSE_CPU)
    MESSAGE(STATUS "We don't appear to be targeting a CPU that we support.")
    MESSAGE(FATAL_ERROR "Please fix this before continuing.")
ENDIF(NOT IODOOM3_CHOSE_CPU)

IF(MACOSX)  # CMake reports CMAKE_SYSTEM_NAME as "Darwin" here...
    SET(IODOOM3_OS "macosx")
ELSE(MACOSX)
    STRING(TOLOWER "${CMAKE_SYSTEM_NAME}" IODOOM3_OS)
ENDIF(MACOSX)

SET(IODOOM3_TARGET "${IODOOM3_OS}-${IODOOM3_CPU}")

IF(IODOOM3_DEBUG_BUILD)
    ADD_DEFINITIONS(-DDEBUG)
    ADD_DEFINITIONS(-D_DEBUG)
ELSE(IODOOM3_DEBUG_BUILD)
    ADD_DEFINITIONS(-D_RELEASE)
    ADD_DEFINITIONS(-DNDEBUG)
    ADD_DEFINITIONS(-DRELEASE)
    IF(CMAKE_COMPILER_IS_GNUCC)
        ADD_DEFINITIONS(-ffast-math)
    ENDIF(CMAKE_COMPILER_IS_GNUCC)
ENDIF(IODOOM3_DEBUG_BUILD)

IF(LINUX)
    # Don't use -rpath (at least, not the silly one CMake forces).
    SET(CMAKE_SKIP_RPATH ON CACHE BOOL "Skip RPATH" FORCE )
    ADD_DEFINITIONS(-Wno-deprecated)
ENDIF(LINUX)

IF(MACOSX)
    IF(IODOOM3_AMD64)
        ADD_DEFINITIONS(-arch x86_64 -mmacosx-version-min=10.6)
        SET(EXE_LDFLAGS "-arch x86_64 -mmacosx-version-min=10.6")
        SET(SHARED_LDFLAGS "-arch x86_64 -mmacosx-version-min=10.6")
    ENDIF(IODOOM3_AMD64)
    IF(IODOOM3_X86)
        ADD_DEFINITIONS(-arch i386 -mmacosx-version-min=10.4)
        SET(EXE_LDFLAGS "-arch i386 -mmacosx-version-min=10.4")
        SET(SHARED_LDFLAGS "-arch i386 -mmacosx-version-min=10.4")
    ENDIF(IODOOM3_X86)
    IF(IODOOM3_PPC)
        ADD_DEFINITIONS(-arch ppc -mmacosx-version-min=10.4)
        SET(EXE_LDFLAGS "-arch ppc -mmacosx-version-min=10.4")
        SET(SHARED_LDFLAGS "-arch ppc -mmacosx-version-min=10.4")
    ENDIF(IODOOM3_PPC)
    SET(OPTIONAL_LIBS ${OPTIONAL_LIBS} "-framework Carbon -framework Cocoa -framework OpenGL -framework OpenAL -framework CoreAudio -framework IOKit")
    ADD_DEFINITIONS(-DMACOS_X=1)
ENDIF(MACOSX)

IF(CMAKE_COMPILER_IS_GNUCC)
    CHECK_C_COMPILER_FLAG("-fno-stack-protector" IODOOM3_GCC_HAS_STACKPROT)
    IF(IODOOM3_GCC_HAS_STACKPROT)
        ADD_DEFINITIONS(-fno-stack-protector)
    ENDIF(IODOOM3_GCC_HAS_STACKPROT)

    CHECK_C_COMPILER_FLAG("-fvisibility=hidden" IODOOM3_GCC_HAS_FVISIBILITY)
    IF(IODOOM3_GCC_HAS_FVISIBILITY)
        ADD_DEFINITIONS(-fvisibility=hidden)
    ENDIF(IODOOM3_GCC_HAS_FVISIBILITY)

    ADD_DEFINITIONS(-pipe -fsigned-char -fno-strict-aliasing -fmessage-length=0)
    ADD_DEFINITIONS(-Wall)

    # !!! FIXME: remove me!
    ADD_DEFINITIONS(-fpermissive)
ENDIF(CMAKE_COMPILER_IS_GNUCC)


# !!! FIXME: libcurl
ADD_DEFINITIONS(-DID_ENABLE_CURL=0)

# !!! FIXME: remove ALSA code altogether.
ADD_DEFINITIONS(-DNO_ALSA)

INCLUDE_DIRECTORIES(
    sound/OggVorbis/ogg
    sound/OggVorbis/oggsrc
    sound/OggVorbis/vorbis
    sound/OggVorbis/vorbissrc
    sound/OggVorbis/vorbissrc
    OpenGL
)

SET(JPG_SRCS
    renderer/jpeg-6/jcapimin.c
    renderer/jpeg-6/jccoefct.c
    renderer/jpeg-6/jccolor.c
    renderer/jpeg-6/jcdctmgr.c
    renderer/jpeg-6/jchuff.c
    renderer/jpeg-6/jcinit.c
    renderer/jpeg-6/jcmainct.c
    renderer/jpeg-6/jcmarker.c
    renderer/jpeg-6/jcmaster.c
    renderer/jpeg-6/jcomapi.c
    renderer/jpeg-6/jcparam.c
    renderer/jpeg-6/jcphuff.c
    renderer/jpeg-6/jcprepct.c
    renderer/jpeg-6/jcsample.c
    renderer/jpeg-6/jdapimin.c
    renderer/jpeg-6/jdapistd.c
    renderer/jpeg-6/jdatadst.c
    renderer/jpeg-6/jdatasrc.c
    renderer/jpeg-6/jdcoefct.c
    renderer/jpeg-6/jdcolor.c
    renderer/jpeg-6/jddctmgr.c
    renderer/jpeg-6/jdhuff.c
    renderer/jpeg-6/jdinput.c
    renderer/jpeg-6/jdmainct.c
    renderer/jpeg-6/jdmarker.c
    renderer/jpeg-6/jdmaster.c
    renderer/jpeg-6/jdmerge.c
    renderer/jpeg-6/jdphuff.c
    renderer/jpeg-6/jdpostct.c
    renderer/jpeg-6/jdsample.c
    renderer/jpeg-6/jdtrans.c
    renderer/jpeg-6/jerror.c
    renderer/jpeg-6/jfdctflt.c
    renderer/jpeg-6/jfdctfst.c
    renderer/jpeg-6/jfdctint.c
    renderer/jpeg-6/jidctflt.c
    renderer/jpeg-6/jidctfst.c
    renderer/jpeg-6/jidctint.c
    renderer/jpeg-6/jidctred.c
    renderer/jpeg-6/jmemmgr.c
    renderer/jpeg-6/jmemnobs.c
    renderer/jpeg-6/jquant1.c
    renderer/jpeg-6/jquant2.c
    renderer/jpeg-6/jutils.c
)

SET(OGGVORBIS_SRCS
    sound/OggVorbis/oggsrc/bitwise.c
    sound/OggVorbis/oggsrc/framing.c
    sound/OggVorbis/vorbissrc/analysis.c
    sound/OggVorbis/vorbissrc/bitrate.c
    sound/OggVorbis/vorbissrc/block.c
    sound/OggVorbis/vorbissrc/codebook.c
    sound/OggVorbis/vorbissrc/envelope.c
    sound/OggVorbis/vorbissrc/floor0.c
    sound/OggVorbis/vorbissrc/floor1.c
    sound/OggVorbis/vorbissrc/info.c
    sound/OggVorbis/vorbissrc/lookup.c
    sound/OggVorbis/vorbissrc/lpc.c
    sound/OggVorbis/vorbissrc/lsp.c
    sound/OggVorbis/vorbissrc/mapping0.c
    sound/OggVorbis/vorbissrc/mdct.c
    sound/OggVorbis/vorbissrc/psy.c
    sound/OggVorbis/vorbissrc/registry.c
    sound/OggVorbis/vorbissrc/res0.c
    sound/OggVorbis/vorbissrc/sharedbook.c
    sound/OggVorbis/vorbissrc/smallft.c
    sound/OggVorbis/vorbissrc/synthesis.c
    sound/OggVorbis/vorbissrc/vorbisenc.c
    sound/OggVorbis/vorbissrc/vorbisfile.c
    sound/OggVorbis/vorbissrc/windowvb.c
)

SET(RENDERER_SRCS
    renderer/Cinematic.cpp
    renderer/GuiModel.cpp
    renderer/Image_files.cpp
    renderer/Image_init.cpp
    renderer/Image_load.cpp
    renderer/Image_process.cpp
    renderer/Image_program.cpp
    renderer/Interaction.cpp
    renderer/Material.cpp
    renderer/MegaTexture.cpp
    renderer/Model.cpp
    renderer/ModelDecal.cpp
    renderer/ModelManager.cpp
    renderer/ModelOverlay.cpp
    renderer/Model_beam.cpp
    renderer/Model_ase.cpp
    renderer/Model_liquid.cpp
    renderer/Model_lwo.cpp
    renderer/Model_ma.cpp
    renderer/Model_md3.cpp
    renderer/Model_md5.cpp
    renderer/Model_prt.cpp
    renderer/Model_sprite.cpp
    renderer/RenderEntity.cpp
    renderer/RenderSystem.cpp
    renderer/RenderSystem_init.cpp
    renderer/RenderWorld.cpp
    renderer/RenderWorld_demo.cpp
    renderer/RenderWorld_load.cpp
    renderer/RenderWorld_portals.cpp
    renderer/VertexCache.cpp
    renderer/cg_explicit.cpp
    renderer/draw_arb.cpp
    renderer/draw_arb2.cpp
    renderer/draw_common.cpp
    renderer/draw_exp_stub.cpp
    renderer/draw_nv10.cpp
    renderer/draw_nv20.cpp
    renderer/draw_r200.cpp
    renderer/tr_backend.cpp
    renderer/tr_deform.cpp
    renderer/tr_font.cpp
    renderer/tr_guisurf.cpp
    renderer/tr_light.cpp
    renderer/tr_lightrun.cpp
    renderer/tr_main.cpp
    renderer/tr_orderIndexes.cpp
    renderer/tr_polytope.cpp
    renderer/tr_render.cpp
    renderer/tr_rendertools.cpp
    renderer/tr_shadowbounds.cpp
    renderer/tr_stencilshadow.cpp
    renderer/tr_subview.cpp
    renderer/tr_trace.cpp
    renderer/tr_trisurf.cpp
    renderer/tr_turboshadow.cpp
)

SET(FRAMEWORK_SRCS
    framework/CVarSystem.cpp
    framework/CmdSystem.cpp
    framework/Common.cpp
    framework/Compressor.cpp
    framework/Console.cpp
    framework/DemoFile.cpp
    framework/DeclAF.cpp
    framework/DeclEntityDef.cpp
    framework/DeclFX.cpp
    framework/DeclManager.cpp
    framework/DeclParticle.cpp
    framework/DeclPDA.cpp
    framework/DeclSkin.cpp
    framework/DeclTable.cpp
    framework/EditField.cpp
    framework/EventLoop.cpp
    framework/File.cpp
    framework/FileSystem.cpp
    framework/KeyInput.cpp
    framework/Unzip.cpp
    framework/UsercmdGen.cpp
    framework/Session_menu.cpp
    framework/Session.cpp
    framework/async/AsyncClient.cpp
    framework/async/AsyncNetwork.cpp
    framework/async/AsyncServer.cpp
    framework/async/MsgChannel.cpp
    framework/async/NetworkSystem.cpp
    framework/async/ServerScan.cpp
)

# !!! FIXME: idlib/bv/Frustum_gcc.cpp is used, unoptimized, on Linux.
# !!! FIXME:  we should remove the #ifdef from Frustum.cpp and see if it's
# !!! FIXME:  still hosed.

SET(IDLIB_SRCS
	idlib/bv/Bounds.cpp
	idlib/bv/Frustum.cpp
	idlib/bv/Sphere.cpp
	idlib/bv/Box.cpp
	idlib/geometry/DrawVert.cpp
	idlib/geometry/Winding2D.cpp
	idlib/geometry/Surface_SweptSpline.cpp
	idlib/geometry/Winding.cpp
	idlib/geometry/Surface.cpp
	idlib/geometry/Surface_Patch.cpp
	idlib/geometry/TraceModel.cpp
	idlib/geometry/JointTransform.cpp
	idlib/hashing/CRC32.cpp
	idlib/hashing/MD4.cpp
	idlib/hashing/MD5.cpp
	idlib/math/Angles.cpp
	idlib/math/Lcp.cpp
	idlib/math/Math.cpp
	idlib/math/Matrix.cpp
	idlib/math/Ode.cpp
	idlib/math/Plane.cpp
	idlib/math/Pluecker.cpp
	idlib/math/Polynomial.cpp
	idlib/math/Quat.cpp
	idlib/math/Rotation.cpp
	idlib/math/Simd.cpp
	idlib/math/Simd_MMX.cpp
	idlib/math/Simd_SSE.cpp
	idlib/math/Simd_SSE2.cpp
	idlib/math/Simd_SSE3.cpp
	idlib/math/Simd_Generic.cpp
	idlib/math/Vector.cpp
	idlib/BitMsg.cpp
	idlib/LangDict.cpp
	idlib/Lexer.cpp
	idlib/Lib.cpp
	idlib/containers/HashIndex.cpp
	idlib/Dict.cpp
	idlib/Str.cpp
	idlib/Parser.cpp
	idlib/MapFile.cpp
	idlib/CmdArgs.cpp
	idlib/Token.cpp
	idlib/Base64.cpp
	idlib/Timer.cpp
	idlib/Heap.cpp
)

SET(CM_SRCS
    cm/CollisionModel_contacts.cpp
    cm/CollisionModel_contents.cpp
    cm/CollisionModel_debug.cpp
    cm/CollisionModel_files.cpp
    cm/CollisionModel_load.cpp
    cm/CollisionModel_rotate.cpp
    cm/CollisionModel_trace.cpp
    cm/CollisionModel_translate.cpp
)

SET(DMAP_SRCS
    tools/compilers/dmap/dmap.cpp
    tools/compilers/dmap/facebsp.cpp
    tools/compilers/dmap/gldraw.cpp
    tools/compilers/dmap/glfile.cpp
    tools/compilers/dmap/leakfile.cpp
    tools/compilers/dmap/map.cpp
    tools/compilers/dmap/optimize.cpp
    tools/compilers/dmap/output.cpp
    tools/compilers/dmap/portals.cpp
    tools/compilers/dmap/shadowopt3.cpp
    tools/compilers/dmap/tritjunction.cpp
    tools/compilers/dmap/tritools.cpp
    tools/compilers/dmap/ubrush.cpp
    tools/compilers/dmap/usurface.cpp
)

SET(AAS_SRCS
    tools/compilers/aas/AASBuild.cpp
    tools/compilers/aas/AASBuild_file.cpp
    tools/compilers/aas/AASBuild_gravity.cpp
    tools/compilers/aas/AASBuild_ledge.cpp
    tools/compilers/aas/AASBuild_merge.cpp
    tools/compilers/aas/AASCluster.cpp
    tools/compilers/aas/AASFile.cpp
    tools/compilers/aas/AASFile_optimize.cpp
    tools/compilers/aas/AASFile_sample.cpp
    tools/compilers/aas/AASReach.cpp
    tools/compilers/aas/AASFileManager.cpp
    tools/compilers/aas/Brush.cpp
    tools/compilers/aas/BrushBSP.cpp
)

SET(ROQ_SRCS
    tools/compilers/roqvq/NSBitmapImageRep.cpp
    tools/compilers/roqvq/codec.cpp
    tools/compilers/roqvq/roq.cpp
    tools/compilers/roqvq/roqParam.cpp
)

SET(RENDERBUMP_SRCS
    tools/compilers/renderbump/renderbump.cpp
)

SET(SND_SRCS
    sound/snd_cache.cpp
    sound/snd_decoder.cpp
    sound/snd_efxfile.cpp
    sound/snd_emitter.cpp
    sound/snd_shader.cpp
    sound/snd_system.cpp
    sound/snd_wavefile.cpp
    sound/snd_world.cpp
)

SET(UI_SRCS
    ui/BindWindow.cpp
    ui/ChoiceWindow.cpp
    ui/DeviceContext.cpp
    ui/EditWindow.cpp
    ui/FieldWindow.cpp
    ui/GameBearShootWindow.cpp
    ui/GameBustOutWindow.cpp
    ui/GameSSDWindow.cpp
    ui/GuiScript.cpp
    ui/ListGUI.cpp
    ui/ListWindow.cpp
    ui/MarkerWindow.cpp
    ui/RegExp.cpp
    ui/RenderWindow.cpp
    ui/SimpleWindow.cpp
    ui/SliderWindow.cpp
    ui/UserInterface.cpp
    ui/Window.cpp
    ui/Winvar.cpp
)

# !!! FIXME: osx, windows
SET(SYS_SRCS
    sys/sys_local.cpp
    sys/posix/posix_net.cpp
    sys/posix/posix_signal.cpp
    sys/posix/posix_threads.cpp
#    sys/stub/util_stub.cpp
)

IF(MACOSX)
    SET(PLATFORM_SYS_SRCS
        sys/osx/DOOMController.mm
        sys/osx/PreferencesDialog.cpp
        sys/osx/macosx_event.mm
        sys/osx/macosx_glimp.mm
        sys/osx/macosx_guids.cpp
        sys/osx/macosx_misc.mm
        sys/osx/macosx_sound.cpp
        sys/osx/macosx_utils.mm
        sys/posix/posix_main.cpp
        sys/linux/stack.cpp
    )
ENDIF(MACOSX)

IF(LINUX)
    SET(PLATFORM_SYS_SRCS
        sys/posix/posix_main.cpp
        sys/linux/stack.cpp
        sys/linux/main.cpp
    )
    SET(PLATFORM_CLIENT_SRCS
        sys/linux/glimp.cpp
        sys/linux/input.cpp
        sys/linux/libXNVCtrl/NVCtrl.c
    )
ENDIF(LINUX)


SET(CLIENT_SRCS
    sys/posix/posix_input.cpp
)

SET(DEDICATED_SRCS
    linux/dedicated.cpp
)

# !!! FIXME: dedicated server needs this:
#-DID_DEDICATED
# (and apparently -DID_GL_HARDLINK, too, but we're ditching that.)

# !!! FIXME: dedicated builds?
SET(IODOOM3_SRCS
    ${JPG_SRCS}
    ${OGGVORBIS_SRCS}
    ${RENDERER_SRCS}
    ${RENDERBUMP_SRCS}
    ${FRAMEWORK_SRCS}
    ${CM_SRCS}
    ${DMAP_SRCS}
    ${SND_SRCS}
    ${UI_SRCS}
    ${SYS_SRCS}
    ${PLATFORM_SYS_SRCS}
    ${IDLIB_SRCS}
    ${AAS_SRCS}
    ${ROQ_SRCS}
    ${CLIENT_SRCS}
    ${PLATFORM_CLIENT_SRCS}
)

ADD_EXECUTABLE(iodoom3
    ${IODOOM3_SRCS}
    ${OPTIONAL_SRCS}
)
SET_TARGET_PROPERTIES(iodoom3 PROPERTIES COMPILE_DEFINITIONS "__DOOM_DLL__")
TARGET_LINK_LIBRARIES(iodoom3 ${OPTIONAL_LIBS} ${IODOOM3_LIBS})
SET_TARGET_PROPERTIES(iodoom3 PROPERTIES LINK_FLAGS "${EXE_LDFLAGS}")

#IF(LINUX)
#    SET_TARGET_PROPERTIES(iodoom3 PROPERTIES LINK_FLAGS "-Wl,-rpath,$ORIGIN ./libSDL-1.2.so.0")
#ENDIF(LINUX)


SET(IODOOM3_GAME_SRCS
	game/AF.cpp
	game/AFEntity.cpp
	game/Actor.cpp
	game/Camera.cpp
	game/Entity.cpp
	game/BrittleFracture.cpp
	game/Fx.cpp
	game/GameEdit.cpp
	game/Game_local.cpp
	game/Game_network.cpp
	game/Item.cpp
	game/IK.cpp
	game/Light.cpp
	game/Misc.cpp
	game/Mover.cpp
	game/Moveable.cpp
	game/MultiplayerGame.cpp
	game/Player.cpp
	game/PlayerIcon.cpp
	game/PlayerView.cpp
	game/Projectile.cpp
	game/Pvs.cpp
	game/SecurityCamera.cpp
	game/SmokeParticles.cpp
	game/Sound.cpp
	game/Target.cpp
	game/Trigger.cpp
	game/Weapon.cpp
	game/WorldSpawn.cpp
	game/ai/AAS.cpp
	game/ai/AAS_debug.cpp
	game/ai/AAS_pathing.cpp
	game/ai/AAS_routing.cpp
	game/ai/AI.cpp
	game/ai/AI_events.cpp
	game/ai/AI_pathing.cpp
	game/ai/AI_Vagary.cpp
	game/gamesys/DebugGraph.cpp
	game/gamesys/Class.cpp
	game/gamesys/Event.cpp
	game/gamesys/SaveGame.cpp
	game/gamesys/SysCmds.cpp
	game/gamesys/SysCvar.cpp
	game/gamesys/TypeInfo.cpp
	game/anim/Anim.cpp
	game/anim/Anim_Blend.cpp
	game/anim/Anim_Import.cpp
	game/anim/Anim_Testmodel.cpp
	game/script/Script_Compiler.cpp
	game/script/Script_Interpreter.cpp
	game/script/Script_Program.cpp
	game/script/Script_Thread.cpp
	game/physics/Clip.cpp
	game/physics/Force.cpp
	game/physics/Force_Constant.cpp
	game/physics/Force_Drag.cpp
	game/physics/Force_Field.cpp
	game/physics/Force_Spring.cpp
	game/physics/Physics.cpp
	game/physics/Physics_AF.cpp
	game/physics/Physics_Actor.cpp
	game/physics/Physics_Base.cpp
	game/physics/Physics_Monster.cpp
	game/physics/Physics_Parametric.cpp
	game/physics/Physics_Player.cpp
	game/physics/Physics_RigidBody.cpp
	game/physics/Physics_Static.cpp
	game/physics/Physics_StaticMulti.cpp
	game/physics/Push.cpp
)
ADD_LIBRARY(game SHARED ${IODOOM3_GAME_SRCS} ${IDLIB_SRCS})
SET_TARGET_PROPERTIES(game PROPERTIES COMPILE_DEFINITIONS "GAME_DLL")
SET_TARGET_PROPERTIES(game PROPERTIES LINK_FLAGS "${SHARED_LDFLAGS}")
SET_TARGET_PROPERTIES(game PROPERTIES INSTALL_NAME_DIR "@executable_path")

SET(IODOOM3_D3XP_SRCS
	d3xp/AF.cpp
	d3xp/AFEntity.cpp
	d3xp/Actor.cpp
	d3xp/Camera.cpp
	d3xp/Entity.cpp
	d3xp/BrittleFracture.cpp
	d3xp/Fx.cpp
	d3xp/GameEdit.cpp
	d3xp/Game_local.cpp
	d3xp/Game_network.cpp
	d3xp/Item.cpp
	d3xp/IK.cpp
	d3xp/Light.cpp
	d3xp/Misc.cpp
	d3xp/Mover.cpp
	d3xp/Moveable.cpp
	d3xp/MultiplayerGame.cpp
	d3xp/Player.cpp
	d3xp/PlayerIcon.cpp
	d3xp/PlayerView.cpp
	d3xp/Projectile.cpp
	d3xp/Pvs.cpp
	d3xp/SecurityCamera.cpp
	d3xp/SmokeParticles.cpp
	d3xp/Sound.cpp
	d3xp/Target.cpp
	d3xp/Trigger.cpp
	d3xp/Weapon.cpp
	d3xp/WorldSpawn.cpp
	d3xp/ai/AAS.cpp
	d3xp/ai/AAS_debug.cpp
	d3xp/ai/AAS_pathing.cpp
	d3xp/ai/AAS_routing.cpp
	d3xp/ai/AI.cpp
	d3xp/ai/AI_events.cpp
	d3xp/ai/AI_pathing.cpp
	d3xp/ai/AI_Vagary.cpp
	d3xp/gamesys/DebugGraph.cpp
	d3xp/gamesys/Class.cpp
	d3xp/gamesys/Event.cpp
	d3xp/gamesys/SaveGame.cpp
	d3xp/gamesys/SysCmds.cpp
	d3xp/gamesys/SysCvar.cpp
	d3xp/gamesys/TypeInfo.cpp
	d3xp/anim/Anim.cpp
	d3xp/anim/Anim_Blend.cpp
	d3xp/anim/Anim_Import.cpp
	d3xp/anim/Anim_Testmodel.cpp
	d3xp/script/Script_Compiler.cpp
	d3xp/script/Script_Interpreter.cpp
	d3xp/script/Script_Program.cpp
	d3xp/script/Script_Thread.cpp
	d3xp/physics/Clip.cpp
	d3xp/physics/Force.cpp
	d3xp/physics/Force_Constant.cpp
	d3xp/physics/Force_Drag.cpp
	d3xp/physics/Force_Field.cpp
	d3xp/physics/Force_Spring.cpp
	d3xp/physics/Physics.cpp
	d3xp/physics/Physics_AF.cpp
	d3xp/physics/Physics_Actor.cpp
	d3xp/physics/Physics_Base.cpp
	d3xp/physics/Physics_Monster.cpp
	d3xp/physics/Physics_Parametric.cpp
	d3xp/physics/Physics_Player.cpp
	d3xp/physics/Physics_RigidBody.cpp
	d3xp/physics/Physics_Static.cpp
	d3xp/physics/Physics_StaticMulti.cpp
	d3xp/physics/Push.cpp
	d3xp/Grabber.cpp
	d3xp/physics/Force_Grab.cpp
)

ADD_LIBRARY(d3xp SHARED ${IODOOM3_D3XP_SRCS} ${IDLIB_SRCS})
SET_TARGET_PROPERTIES(d3xp PROPERTIES COMPILE_DEFINITIONS "_D3XP;CTF;GAME_DLL")
SET_TARGET_PROPERTIES(d3xp PROPERTIES LINK_FLAGS "${SHARED_LDFLAGS}")
SET_TARGET_PROPERTIES(d3xp PROPERTIES INSTALL_NAME_DIR "@executable_path")

# end of CMakeLists.txt ...

