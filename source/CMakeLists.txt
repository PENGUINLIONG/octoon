# contrib folders
INCLUDE_DIRECTORIES(${DEPENDENCIES_PATH}/zlib)
INCLUDE_DIRECTORIES(${DEPENDENCIES_PATH}/glew)

# octoon folder
INCLUDE_DIRECTORIES(${PROJECT_SOURCE_DIR}/include)

# lib folder for link
LINK_DIRECTORIES(${LIBRARY_OUTPUT_PATH})

ADD_SUBDIRECTORY(runtime)
ADD_SUBDIRECTORY(engine)