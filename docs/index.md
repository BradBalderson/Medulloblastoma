# Index
Index which describes each script in scripts/ in terms of INPUT/OUTPUT, and 
brief description of function.

X1_QC_SpeciesClassify/

    X1_QC.py -> Performs the QC to filter spots & looks at the distribution of 
                                                               the counts/genes.
             INPUT:  * data/Visium8_{sample_name}_*/*
             OUTPUT: * data/filter_ids/{sample_name}_filtered.txt -> 
                              Contains the barcodes of the spots that passed QC.       
                     * figure_components/QC_figure/*
                     
    X2_generate_seurat_rds.R -> Generates seurat rds with sctransform normalised
                                data which is later used for species scoring &
                                giotto enrichment analysis.
                                NOTE: memory intensive, had to run on a hpc.
                                
                    INPUT: * data/Visium8_{sample_name}_*/*
                           * data/filter_ids/{sample_name}_filtered.txt
                           
                    OUTPUT: * seurat_rds/*
                    
    X3_human_mouse_score.R -> Sums to sctransformed normalised gene expression
                             for human/mouse genes to generate human/mouse 
                             scores per spot in each sample, to subsequently
                             allow for species clasification.
                             
                    INPUT: * seurat_rds/*treated.rds
                    OUTPUT: * spot_meta/*_species.txt
                    
    X4_species_classify.py -> Uses the species scores to classify spots by into
                                                                human/mix/mouse.
                                    
                     INPUT: * data/Visium8_{sample_name}_*/*
                            * spot_meta/*_species.txt
                     OUTPUT: * data/spot_meta/species_classify_v2/
                             * figure_components/species_figure/
                             
X2_DEAnalysis/

    X1_Pseudo_Limma_DE_human.R -> Performs the DE between Palbociclib treated & 
                                 control samples pseudobulked human spots/genes.
                            
                            INPUT: * data/seurat_rds/all.rds
                                   * data/spot_meta/species_classify_v2/
                                                                   *_species.txt
                            
                            OUTPUT: * data/DE_out/Pseudo_Limma_human/*
                            
    X2_Pseudo_Limma_DE_mixHuman.R -> Performs the DE between Palbociclib treated 
                           & control samples pseudobulked mix spots/human genes.
    
                            INPUT: * data/seurat_rds/all.rds
                                   * data/spot_meta/species_classify_v2/
                                                                   *_species.txt
                            
                            OUTPUT: * data/DE_out/Pseudo_Limma_mixHuman/*
    
    X3_Pseudo_Limma_DE_mixMouse.R -> Performs the DE between Palbociclib treated 
                           & control samples pseudobulked mix spots/mouse genes.
                           
                           INPUT: * data/seurat_rds/all.rds
                                  * data/spot_meta/species_classify_v2/
                                                                   *_species.txt
                            
                           OUTPUT: * data/DE_out/Pseudo_Limma_mixMouse/*
    
    X4_Pseudo_Limma_DE_mouse.R -> Performs the DE between Palbociclib treated & 
                                 control samples pseudobulked mouse spots/genes.
                                 
                             INPUT: * data/seurat_rds/all.rds
                                    * data/spot_meta/species_classify_v2/
                                                                   *_species.txt
                            
                             OUTPUT: * data/DE_out/Pseudo_Limma_mouse/*
                           
    X5_clusterProfiler_human.R -> Performs the GSEA on the human DE genes ranked
                                    by t-value.       
                             
                             INPUT: * data/DE_out/Pseudo_Limma_human/
                                               de_results_PseudoLimma_human.xlsx
                             OUTPUT: * data/DE_out/Pseudo_Limma_human/gsea_out/*
                                     * figure_components/DE_figures/
                                                          _human_human_GSEA*.pdf
                                                          
    X6_DE_figures.py -> Creates volcano plots/MDS plots to 
                        visualise the DE results for the human/mouse/mix. 
                        Also includes the PCA analysis.
                        
                        INPUT:  data/DE_out/Pseudo_Limma*/de_results_*
                        
                        OUTPUT: figure_components/DE_figures/*
                                data/DE_out/Table_S2_MixSpotDE.xlsx
                    
X3_GiottoEnrichment/

    X1_giotto_spot_enrich.R -> Using the Giotto package for PAGE per-spot 
                                                            enrichment analysis:
                    https://genomebiology.biomedcentral.com/articles/10.1186/s13059-021-02286-2#Sec2
                    http://spatialgiotto.rc.fas.harvard.edu/giotto.install.native.html#s_macos
                    
                    To perform enrichment for DE E2F targets & different 
                    MB-SHH transcriptional programs, which were downloaded from
                    Hovestadt, et al. into data/third_party_data/.
                    
                    INPUT: * data/seurat_rds/all.rds
                           * data/Pseudo_Limma_human/gsea_out/
                                                         gsea_results_human.xlsx
                           * data/third_party_data/*_program.txt
                           
                    OUTPUT: * data/giotto_out/*
      
    X2_giotto_figures.py -> Creates the spatial plots of the per-spot enrichment
                          in a consistent way across the samples for comparison.
                          
                    INPUT: * data/Visium8_{sample_name}_*/*
                           * data/filter_ids/*
                           * data/giotto_out/SHH_gsea_v2_scores.txt
                           
                    OUTPUT: * figure_components/giotto_figures/*
                    
    X3_SHH_violins.py -> Creates violin plots showing the shift of the 
                              different cell types in response to the treatment. 
                              
                         INPUT: * data/giotto_out/SHH_gsea_v2_scores.txt
                              
                         OUTPUT: * figure_components/giotto_figures/*
                            
X4_HumanSpotAnnotation/

    X1_SME_normalise.py -> Loads in the data, performs SME normalisation, &
                            saves output.
                            
                            INPUT: * data/Visium8_{sample_name}_*/*
                            OUTPUT: * data/scanpy_h5ads/*_all_species_SME.h5ad

    X2_FetalBrain3_SingleR.R -> Loads in the fetal brain data, & uses this as a
                                reference to label cells by dominant cell type.
                    First download the fetal human brain data to match INPUT:
                    Ref. paper: https://www.nature.com/articles/s41586-020-2157-4
                    Download link: https://db.cngb.org/HCL/gallery.html?tissue=Fetal-Brain3

                    INPUT: * data/scanpy_h5ads/*_all_species_SME.h5ad
                           * data/third_party_data/HCL2020/Fetal-Brain3_dge.txt
                           * data/third_party_data/HCL2020/Fetal-Brain3_Anno.txt
                           
                    OUTPUT: * data/spot_meta/*FetalBrain3singleR_scores.txt
                            * figure_components/HumanAnnot_figures/

    X3_FetalBrain3_umap.py -> Loads in the fetal brain data, & UMAP from seurat 
                    to display the data in a figure.
                  Ref. paper: https://www.nature.com/articles/s41586-020-2157-4
                  Download link: https://db.cngb.org/HCL/gallery.html?tissue=Fetal-Brain3
                  
                  INPUT: * data/third_party_data/HCL2020/Fetal-Brain3_dge.txt
                         * data/third_party_data/HCL2020/Fetal-Brain3_Anno.txt
                         
                  OUTPUT: * data/third_party_data/HCL2020/FetalBrain3.h5ad
                          * figure_components/HumanAnnot_figures/*

    X4_singleR_humanLabel_panels.py -> Label the data according to the human 
                        labels from the Fetal Human 3 reference & output figure.
                      
                      INPUT: * data/scanpy_h5ads/*_all_species_SME.h5ad
                             * data/spot_meta/species_classify_v2/
                                           *FetalBrain3_singleR_scores_human.txt
                                           
                      OUTPUT: * figure_components/HumanAnnot_figures/
                                                  *FetalBrain3Labels_spatial.pdf
    
    
X5_MouseSpotAnnotation/

    X1_format_Vladoiu.py -> Compiles the dev. mouse cerebellum from
                            Vladiou, et al. to use a reference to annotate mouse
                            spots later. Very memory intensive; ran on HPC. 
                         
                            Need to download the supplementary count data from 
                            GEO using the http link and untar so matches the 
                            INPUT description:
                            Ref. Paper: https://www.nature.com/articles/s41586-019-1158-7
                            Data link: https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE118068
                            
                            INPUT: * data/third_party_data/
                                         Vladiou2019_Nature_scRNA/GSE118068_RAW/
                            
                            OUTPUT: * data/third_party_data/
                                            Vladiou2019_Nature_scRNA/
                                                     Vladoiu2019_lateRef.h5ad -> 
                                                             Just P7 cell types.
                                                             
    X2_Vladoiu_SingleR.R -> Performs the automated annotation of the mouse spots
                            using the Vladoiu mouse cerbellum data as a reference.
                            
                            INPUT: * data/scanpy_h5ads/*_all_species_SME.h5ad
                                   * data/third_party_data/
                                              Vladiou2019_Nature_scRNA/
                                                        Vladoiu2019_lateRef.h5ad
                                                        
                            OUTPUT: * data/spot_meta/species_classify_v2/
                                               *Vladoiu_singleR_scores_mouse.txt
                                    * figure_components/MouseAnnot_figures/   
                                    
    X3_singleR_mouseLabel_panels.py -> Label the data according to the 
                                  mouse labels from the Vladiou & output figure.
                          
                          INPUT: * data/scanpy_h5ad/*_all_species_SME.h5ad
                                 * data/spot_meta/species_classify_v2/
                                               *Vladoiu_singleR_scores_mouse.txt
                          OUTPUT: * figure_components/MouseAnnot_figures/
                                                         *VladLabels_spatial.pdf
                                                         
    X4_border_enrichment.py -> Perform FET to test for enrichment of
                        dominant spot cell types for the border spots to provide
                        statistical evidence for astrocytes at the border.

                        INPUT: * data/Visium8_{sample_name}_*/*
                               * data/scanpy_h5ad/*_all_species_SME.h5ad
                               * data/spot_meta/species_classify_v2/
                                               *Vladoiu_singleR_scores_mouse.txt

                        OUTPUT: * data/spot_meta/astro_enrich_stats.xlsx  
				* data/scanpy_h5ads/integrated_border.h5ad
                                * figure_components/MouseAnnot_figures/
                                                             *border_spatial.pdf
                        
    X5_gfap_panels.py -> Generating gene plot panels to show GFAP 
                                 expression as prognostic for astrocytes. 
                                 
                       INPUT: data/scanpy_h5ad/*_all_species_SME.h5ad
                              data/spot_meta/species_classify_v2/
                                               *Vladoiu_singleR_scores_mouse.txt
                       OUTPUT: figure_components/MouseAnnot_figures/
                                                               *Gfap_spatial.pdf                                           
X6_CellCellInteraction/

	X1_mouse_LR-CCI.ipynb -> Runs the stlearn LR interaction analysis for mouse genes, 
				finds LRs which have spots on the border, looks at how these
				are identified across samples, & then performs
				cell type permutation to see which cell types are 
				interacting via these LR pairs.

			INPUT: * data/scanpy_h5ads/MB_*_all-mouse_LRResults.h5ad
               	               * data/scanpy_h5ads/integrated_border.h5ad
               		       * data/spot_meta/species_classify_v2/*_Vladoiu_singleR_scores_mouse.txt
        		OUTPUT: * data/cci/mouse/*_border_enriched_LRs.txt
                		* data/cci/mouse/interface_overlaps.xlsx	

	X2_human_LR-CCI.ipynb ->  Runs the stlearn LR interaction analysis for mouse genes,
                                finds LRs which have spots on the border, & then performs
                                cell type permutation to see which cell types are
                                interacting via these LR pairs.

			INPUT: * data/scanpy_h5ads/MB_*_all-human_LRResults.h5ad
                               * data/scanpy_h5ads/integrated_border.h5ad
                               * data/spot_meta/species_classify_v2/*_FetalBrain3_singleR_scores_human.txt
                        OUTPUT: * data/cci/human/*_border_enriched_LRs.txt			

    X3_clusterProfiler_mouse_LR-analysis.R -> Using cluster profiler to do an 
                over-representation analysis on the mouse LR pairs with atleast 
                10 significant spots on the tumour border. But this time adding 
                them all to the same gene list, instead of performing 
                                                  independently for each sample.

             INPUT:  * data/cci/mouse/interface_overlaps.xlsx
             OUTPUT: * figure_components/cci_figures/_mouse_LR-genes_GSEAemap.pdf
                     * data/cci/mouse/gsea_results_mouse_LR-genes.xlsx

	    
    X4_clusterProfiler_human_LR-analysis.R -> # Using cluster profiler to do an 
                over-representation analysis on the LR pairs with atleast 4 
                significant spots on the tumour border.

            INPUT:  * data/cci/human/*_border_enriched_LRs.txt
            OUTPUT: * figure_components/cci_figures/_human_LR-genes_GSEAemap.pdf
                    * data/cci/mouse/gsea_results_human_LR-genes.xlsx

X7_revision-analysis/
    -> Analysis conducted during revision process, performed by 
        Dr. Guiyan Ni, and Onkar Mulay.
        
     X1_RNAScope_Unlog_MKI67_vs_Gfap.ipynb -> Is actuallly an R jupyter notebook.
                                Contains the analysis of the RNAScope data, used
                                to validate increased expression of astrocytes,
                                microglia, and proliferation markers at the 
                                tumour border.
                                -> Performed by Onkar Mulay.
                                    * https://www.linkedin.com/in/onkar-mulay-25099a157/?originalSubdomain=in
     
     X2_athena_plot.nb.html -> R markdown html. 
                                Uses entropy of gene expression per spot, 
                                between control and Palbociclib treated ST-seq 
                                samples, to show reduction in cellular 
                                heterogeneity in the treatment condition. 
                                -> Performed by Dr. Guiyan Ni.
                                    * https://www.linkedin.com/in/guiyan-ni-3698851a3/?originalSubdomain=au
                                
        
        
                
