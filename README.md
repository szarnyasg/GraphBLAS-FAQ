# GraphBLAS-FAQ

## Identifiers

* **Question:** My vertices have non-contiguous identifiers. How do I turn my graph into matrices?

* **Answer:** There are two major directions to tackle this:

    1. **Relabelling.** A possible solution is remapping the sparse identifiers to dense ones. This problem is known as *dense vertex relabelling* and is challenging to solve efficiently. The LAGraph library support [dense relabelling](https://github.com/GraphBLAS/LAGraph/blob/a627cefff60e2ea4ae4701e7a48c1353bf490dfe/Experimental/Algorithm/LAGraph_dense_relabel.c) using GraphBLAS constructs.

       * **Tradeoffs:**
         - Implementing an efficient relabelling method is challenging.
         - Once the processing finished, it is often desirable to map the results back to the original data set.
         - The processing phase (running the graph algorithms) is typically faster as the underlying GraphBLAS library can use more multiplication algorithms for regular sparse (non-hypersparse) matrices.

    2. **Hypersparse matrices.** Using SuiteSparse:GraphBLAS, it is possible to **leave the identifiers as-is**. The library will detect that it should build a [hypersparse matrix](https://people.eecs.berkeley.edu/~aydin/hypersparse-ipdps08.pdf). Hypersparse representations use a variant of the CSR/CSC compression techniques.

       * **Tradeoffs:**
         - Loading will be faster at the cost of a small increase in processing times. If the processing time is not too high, the overall performance might even be better.
         - The code will be simpler as mapping between sparse-dense identifiers will no longer be necessary (during load or when mapping the results back to the original code).

       * **Open problem:** SuiteSparse:GraphBLAS only supports matrices up to (2^60-1) × (2^60-1), making hypersparse matrices only suitable for graphs where the vertex ids as smaller than 2^60. What happens when the user has ids up to 2^64-1?
