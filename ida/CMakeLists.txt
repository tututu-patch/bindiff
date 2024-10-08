# Copyright 2011-2024 Google LLC
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     https://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.


# TODO(cblichmann): Remove this workaround for macOS protobuf build
if(APPLE)
  add_definitions(-DGOOGLE_PROTOBUF_NO_RDTSC)
endif()

# Plugins will be named "bindiffX_ida.so" and "bindiffX_ida64.so" (or .dll or
# .dylib), where X is the release number.
add_ida_plugin(${bindiff_ida_plugin_name}
  bindiff_icon.cc
  bindiff_icon.h
  main_plugin.cc
  main_plugin.h
  matched_functions_chooser.cc
  matched_functions_chooser.h
  names.cc
  names.h
  results.cc
  results.h
  statistics_chooser.cc
  statistics_chooser.h
  unmatched_functions_chooser.cc
  unmatched_functions_chooser.h
  visual_diff.cc
  visual_diff.h
)
ida_target_link_libraries(${bindiff_ida_plugin_name}
  binexport_core
  binexport_plugin_shared
  bindiff_base
  bindiff_shared
  absl::cleanup
  absl::function_ref
  absl::memory
  absl::nullability
  absl::str_format
  absl::time
)
if(WIN32)
  ida_target_link_libraries(${bindiff_ida_plugin_name}
    delayimp.lib  # Helper for /DELAYLOAD
    ws2_32.lib
  )
  # See b/206656063 for context
  set_ida_target_properties(${bindiff_ida_plugin_name} PROPERTIES
    LINK_FLAGS /DELAYLOAD:dbghelp.dll
  )
endif()
set_ida_target_properties(${bindiff_ida_plugin_name} PROPERTIES
  POSITION_INDEPENDENT_CODE ON
  INTERPROCEDURAL_OPTIMIZATION ${BINDIFF_ENABLE_IPO}
)
ida_install(TARGETS ${bindiff_ida_plugin_name}
            ARCHIVE DESTINATION bindiff-prefix
            RUNTIME DESTINATION bindiff-prefix
            LIBRARY DESTINATION bindiff-prefix)
