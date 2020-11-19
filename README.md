# GraphBLAS FAQ

## Using GraphBLAS

### This is textbook stuff, why isn't is used more widely?

While the applicability of semirings and sparse matrices for formulating graph algorithms has been [well-known](https://www.goodreads.com/book/show/112266.The_Design_and_Analysis_of_Computer_Algorithms), it was not widely exploited.
Naturally, the reasons behind this are manifold but I believe the key reason is that the linear algebraic formulation is shines best when the operations can be executed in parallel and there were no easy-to-use parallel libraries until very recently.
While there were some librariers supporting parallel matrix operations (e.g. the Combinatorial BLAS), the first library that can be installed in minutes is SuiteSparse:GraphBLAS v3.2.0 which was released in late 2019.

### So is GraphBLAS ready for production use?

*My opinion* is the following:

* SuiteSparse:GraphBLAS is a production-ready library. Other open-source libraries are experimental, not geared towards high performance, etc.
* LAGraph is an experimental library and not ready for production use.

## Design patterns

### Loading graphs with non-contiguous vertex identifiers

* **Question:** My vertices have non-contiguous identifiers. How do I turn my graph into matrices?

* **Answer:** There are two major directions to tackle this:

    1. **Relabelling.** A possible solution is remapping the sparse identifiers to dense ones. This problem is known as *dense vertex relabelling* and is challenging to solve efficiently. The LAGraph library support [dense relabelling](https://github.com/GraphBLAS/LAGraph/blob/a627cefff60e2ea4ae4701e7a48c1353bf490dfe/Experimental/Algorithm/LAGraph_dense_relabel.c) using GraphBLAS constructs.

       * **Tradeoffs:**
         - Implementing an efficient relabelling method is challenging.
         - Once the processing finished, it is often desirable to map the results back to the original data set.
         - The processing phase (running the graph algorithms) is typically faster as the underlying GraphBLAS library can use more multiplication algorithms for regular sparse (non-hypersparse) matrices.

    2. **Hypersparse matrices.** Using SuiteSparse:GraphBLAS, it is possible to **leave the identifiers as-is**. The library will detect that it should build a [hypersparse matrix](https://people.eecs.berkeley.edu/~aydin/hypersparse-ipdps08.pdf). Hypersparse representations use a variant of the CSR/CSC compression techniques.

       * **Tradeoffs:**
         - Loading will be faster at the cost of a small increase in processing times. If the processing time is not too high compared to the loading time, the overall performance might even be better.
         - The code will be simpler as mapping between sparse-dense identifiers will no longer be necessary (during load or when mapping the results back to the original code).

       * **Open problem:** SuiteSparse:GraphBLAS only supports matrices up to (2^60-1) × (2^60-1), making hypersparse matrices only suitable for graphs where the vertex ids are smaller than 2^60. What happens when the user has ids up to 2^64-1?

### Projecting a subgraph for the same rows & columns

Don't care about the dimensions of the matrices?

1. Use `GrB_extract` to get the small matrix. For subsequent operations (e.g. `GrB_mxm`), also use extract along the corresponding dimension(s).

Want to keep the dimensions?

1. `GrB_extract` to a smaller matrix + use GrB_assign to restore the dimensions of the original matrix
2. Multiply with a selection matrix from the left and from the right (`S*A*S`)
3. Use a dense vector to encode the vertices and `GxB_select` (see [discussion](https://github.com/GraphBLAS/LAGraph/issues/83))

### Handling edge types

1. single matrix with values
2. T boolean matrices
### Handling node labels

1. base it on the edges
2. use a label matrix

### Handling properties

TODO.
