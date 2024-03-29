
Manual
======

.. contents:: Contents
   :depth: 2
   :local:


Input:

#. Indexed BAM file for 10x scRNA-seq data;
#. Phased SNPs;
#. Clone-specific CNV profile;
#. Cells in each clone;

Output:

#. In stage 1, the cell x region x allele UMI list;
#. In stage 2, new indexed BAM file;


Input
-----

Phased SNPs
~~~~~~~~~~~
The input phased SNPs should be stored in either a tab delimited text file
(i.e., TSV file) or a VCF file.


Input is in TSV format
++++++++++++++++++++++

If it is in a TSV file, then the first 6 columns of the file should be
``chrom``, ``pos``, ``ref``, ``alt``, ``ref_hap``, ``alt_hap``, where

chrom : str
    The chromosome name, e.g., chr1.

pos : int
    The genomic position of the SNP, 1-based.

ref : str
    The reference (REF) allele of the SNP, one of ``{'A', 'C', 'G', 'T'}``.

alt : str
    The alternative (ALT) allele of the SNP, one of ``{'A', 'C', 'G', 'T'}``.

ref_hap : int
    The haplotype index of the reference (REF) allele, one of ``{0, 1}``.

alt_hap : int
    The haplotype index of the alternative (ALT) allele, one of ``{0, 1}``.

Note that **this input file should not contain a header line**. 
An example is as follows:

.. code-block::

  chr1     120    A     C       0       1
  chr1     260    C     T       1       0
  chr2     580    A     G       1       0


Input is in VCF format
++++++++++++++++++++++

If it is in VCF format, the file should contain the ``GT`` field in 
``FORMAT`` (i.e., the 9th column).
The corresponding phased genotype could be delimited by either ``'/'`` or
``'|'``, e.g., "0/1", or "0|1".

.. note::
   As reference phasing, e.g., with Eagle2, is not perfect, one UMI may 
   cover two SNPs with conflicting haplotype states.
   Additionally, reference phasing tends to have higher rate in longer 
   distance.
   Therefore, further local phasing (e.g., in gene level) and global phasing
   (e.g., in bin level) could be used to reduce error rate, e.g., with the
   3-step phasing used by CHISEL_ in scDNA-seq data and XClone_ in scRNA-seq
   data.
   

Clone-specific CNV profile
~~~~~~~~~~~~~~~~~~~~~~~~~~
The clone-specific CNV profile should be stored in a TSV file
with the first 7 columns being ``chrom``, ``start``, ``end``, ``region_id``,
``clone_id``, ``cn_ale0``, ``cn_ale1``, where

chrom : str
    The chromosome name, e.g., chr1.

start : int
    The start genomic position of the CNV region, 1-based and inclusive.

end : int
    The end genomic position of the CNV region, 1-based and inclusive.
    To specify the end of the whole chromosome, you can use either the actual
    genomic position or simply ``Inf``.

region_id : str
    The ID of the CNV region, can be arbitrary string as long as it is unique
    in each clone.

clone_id : str
    The clone ID.

cn_ale0 : int
    The copy number of the first allele (haplotype).

cn_ale1 : int
    The copy number of the second allele (haplotype).

Each clone-specific CNV per line.

Note that **this input file should not contain a header line**. 
An example is as follows:

.. code-block::

   chr1   1000    28000   chr1:1000-28000     clone_0   1    2
   chr1   28001   69000   chr1:28001-69000    clone_0   2    1
   chr2   320000  560000  chr2:320000-560000  clone_0   0    1
   chr2   320000  560000  chr2:320000-560000  clone_1   1    0
   chr6   18000   68000   chr6:18000-68000    clone_1   2    0
   chr18  1       Inf     chr18               clone_2   2    2

By specifying different values for ``cn_ale0`` and ``cn_ale1``, you may
mimic various CNV types, including copy gain (e.g., setting ``1, 2``), 
copy loss (e.g., setting ``0, 1``), LOH (e.g., setting ``2, 0``).

This format fully support allele-specific CNVs.
For instance, to simulate the scenario that two subclones have copy loss in
the same region while on distinct alleles, setting ``cn_ale0, cn_ale1``
to ``0, 1`` and ``1, 0`` in two subclones, respectively, as the
above example shows.

It also supports WGD, e.g., by setting ``cn_ale0, cn_ale1`` of all 
chromosomes to ``2, 2``.
Generally, detecting WGD from scRNA-seq data is challenging, as it is hard
to distinguish WGD from high library size.
One scenario eaiser to detect WGD is that a balanced copy loss occurred 
after WGD, e.g., ``cn_ale0, cn_ale1`` of chr3 and other chromosomes are
``1, 1`` and ``2, 2`` respectively.
In this case, chr3 may have signals of balanced BAF while copy-loss RDR,
which should not happen on normal diploid genome.


Cells in each clone
~~~~~~~~~~~~~~~~~~~
The barcodes of cells in each CNV clone should be stored in a TSV file with
the first 2 columns being ``cell_barcode`` and ``clone_id``, where

cell_barcode : str
    The cell barcode, typically under the ``CB`` tag in 10x BAM file.

clone_id : str
    The clone ID.

Note that **this input file should not contain a header line**. 
An example is as follows:

.. code-block::

   AAAAACGTACGTAAAA-1   clone_0
   ACGTAAAAAGGTACGT-1   clone_0
   ACGTACGTATGTAAAA-1   clone_0
   ACGTAGGTACGTAACA-1   clone_1
   ACGTAGTTACGTATAC-1   clone_1
   AGCTCCGTACGTAAGA-1   clone_2
   AGGTGCGTACGTGCAT-1   clone_2


Output
------

The cell x region x allele UMI list
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
The extracted cell x region x allele UMI list is stored in a TSV file with
the first 4 columns being ``cell_barcode``, ``region_id``, ``UMI``, and
``allele``, where

cell_barcode : str
    The cell barcode.

region_id : str
    The ID of the CNV region, typically concatenating the chromosome name,
    the start and end positions of the region, e.g., "chr1:1000-28000",
    "chr18" (the whole chr18), or "chr12:100" (region from chr12:100 to the
    end of the chr12).

UMI : str
    The UMI barcode.

allele : int
    The index of the allele/haplotype, one of {0, 1}.

Note that **this input file should not contain a header line**. 
An example is as follows:

.. code-block::

   AAAAACGTACGTAAAA-1   chr1:1000-28000 AAGTACGTACGT    0
   AAAAACGTACGTAAAA-1   chr1:1000-28000 ACGTACGTACGT    1
   AAAAACGTACGTAAAA-1   chr1:1000-28000 AGGTACGTACGT    1
   AAAAACGTACGTAAAA-1   chr18   ACGTAGGTACGT    0
   AAAAACGTACGTAAAA-1   chr18   ACGTATGTACGT    0


.. _CHISEL: https://www.nature.com/articles/s41587-020-0661-6
.. _XClone: https://www.biorxiv.org/content/10.1101/2023.04.03.535352v2

