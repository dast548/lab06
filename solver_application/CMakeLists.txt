cmake_minimum_required(VERSION 3.4)
project(solver_app)

add_executable(solver_app equation.cpp)

target_link_libraries(solver_app formatter_ex solver)
