cmake_minimum_required(VERSION 3.0.0)
project(grpcasync VERSION 0.1.0)

include(CTest)
enable_testing()

set(THREADS_PREFER_PTHREAD_FLAG ON)
find_package(Protobuf REQUIRED)
find_package(Threads REQUIRED)
find_package(gRPC CONFIG REQUIRED)
if (gRPC_FOUND)
   Message("Find grpc package")
   set(GRPC_REFLECTION gRPC::grpc++_reflection)
else()
   Message("Not find grpc package")
   set(GRPC_REFLECTION grpc++_reflection)
endif()

#set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")
include_directories(/usr/local/include)
link_directories(/usr/local/lib)

set(PROTO_PATH "${CMAKE_SOURCE_DIR}")
set(HELLOWORLD_PROTO "${PROTO_PATH}/helloworld.proto")
set(HELLOSTREAMINGWORLD_PROTO "${PROTO_PATH}/hellostreamingworld.proto")
set(GENERATED_PROTOBUF_PATH "${CMAKE_BINARY_DIR}/generated")
file(MAKE_DIRECTORY ${GENERATED_PROTOBUF_PATH})

set(HELLOWORLD_PB_CPP_FILE "${GENERATED_PROTOBUF_PATH}/helloworld.pb.cc")
set(HELLOWORLD_PB_H_FILE "${GENERATED_PROTOBUF_PATH}/helloworld.pb.h")
set(HELLOWORLD_GRPC_PB_CPP_FILE "${GENERATED_PROTOBUF_PATH}/helloworld.grpc.pb.cc")
set(HELLOWORLD_GRPC_PB_H_FILE "${GENERATED_PROTOBUF_PATH}/helloworld.grpc.pb.h")

set(HELLOSTREAMINGWORLD_PB_CPP_FILE "${GENERATED_PROTOBUF_PATH}/hellostreamingworld.pb.cc")
set(HELLOSTREAMINGWORLD_PB_H_FILE "${GENERATED_PROTOBUF_PATH}/hellostreamingworld.pb.h")
set(HELLOSTREAMINGWORLD_GRPC_PB_CPP_FILE "${GENERATED_PROTOBUF_PATH}/hellostreamingworld.grpc.pb.cc")
set(HELLOSTREAMINGWORLD_GRPC_PB_H_FILE "${GENERATED_PROTOBUF_PATH}/hellostreamingworld.grpc.pb.h")


add_custom_command(
    OUTPUT "${HELLOWORLD_PB_H_FILE}"
           "${HELLOWORLD_PB_CPP_FILE}"
           "${HELLOWORLD_GRPC_PB_H_FILE}"
           "${HELLOWORLD_GRPC_PB_CPP_FILE}"
    COMMAND ${PROTOBUF_PROTOC_EXECUTABLE}
    ARGS "--proto_path=${PROTO_PATH}"
         "--cpp_out=${GENERATED_PROTOBUF_PATH}"
         "${HELLOWORLD_PROTO}"
    COMMAND ${PROTOBUF_PROTOC_EXECUTABLE}
    ARGS "--proto_path=${PROTO_PATH}"
         "--grpc_out=${GENERATED_PROTOBUF_PATH}"
         "--plugin=protoc-gen-grpc=/usr/local/bin/grpc_cpp_plugin"
         "${HELLOWORLD_PROTO}"
    )

add_custom_command(
    OUTPUT "${HELLOSTREAMINGWORLD_PB_H_FILE}"
           "${HELLOSTREAMINGWORLD_PB_CPP_FILE}"
           "${HELLOSTREAMINGWORLD_GRPC_PB_H_FILE}"
           "${HELLOSTREAMINGWORLD_GRPC_PB_CPP_FILE}"
    COMMAND ${PROTOBUF_PROTOC_EXECUTABLE}
    ARGS "--proto_path=${PROTO_PATH}"
         "--cpp_out=${GENERATED_PROTOBUF_PATH}"
         "${HELLOSTREAMINGWORLD_PROTO}"
    COMMAND ${PROTOBUF_PROTOC_EXECUTABLE}
    ARGS "--proto_path=${PROTO_PATH}"
         "--grpc_out=${GENERATED_PROTOBUF_PATH}"
         "--plugin=protoc-gen-grpc=/usr/local/bin/grpc_cpp_plugin"
         "${HELLOSTREAMINGWORLD_PROTO}"
    )



set(GENERATED_PROTOBUF_FILES ${HELLOWORLD_PB_H_FILE} ${HELLOWORLD_PB_CPP_FILE}
${HELLOWORLD_GRPC_PB_H_FILE} ${HELLOWORLD_GRPC_PB_CPP_FILE})

set(GENERATED_PROTOBUF_STREAMING_FILES ${HELLOSTREAMINGWORLD_PB_H_FILE} ${HELLOSTREAMINGWORLD_PB_CPP_FILE}
${HELLOSTREAMINGWORLD_GRPC_PB_H_FILE} ${HELLOSTREAMINGWORLD_GRPC_PB_CPP_FILE})

include_directories(${GENERATED_PROTOBUF_PATH})
foreach(target greeter_async_client greeter_async_server greeter_client greeter_server)
    add_executable(${target} ${CMAKE_CURRENT_SOURCE_DIR}/${target}.cc ${GENERATED_PROTOBUF_FILES})
    target_link_libraries(${target} ${GRPC_REFLECTION} grpc++  ${PROTOBUF_LIBRARY})
endforeach()


foreach(target greeter_streaming_client async_streaming_server_alarm 
               async_streaming_server_queue_to_back async_streaming_server_queue_to_front 
               async_streaming_server)
    add_executable(${target} ${CMAKE_CURRENT_SOURCE_DIR}/${target}.cc ${GENERATED_PROTOBUF_STREAMING_FILES})
    target_link_libraries(${target} ${GRPC_REFLECTION} grpc++  ${PROTOBUF_LIBRARY})
endforeach()



