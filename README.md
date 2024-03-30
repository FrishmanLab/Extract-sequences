



#!/bin/bash
#SBATCH --job-name=Pig
#SBATCH --output=Pig.out
#SBATCH --error=Pig.err
#SBATCH --partition=GPU-2080
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=28
#SBATCH --mem=50G

Work_Dir="/home/students/q.abbas/pig"
genome="/home/students/q.abbas/pig/"
proteindb="/proj/q.abbas/databases/Proteomics/metazoa/OrthoDB_Metazoa.fa"
Species="pig"
mkdir -p $Work_Dir/braker
mkdir -p $Work_Dir/braker/default
mkdir -p $Work_Dir/braker/seeds
mkdir -p $Work_Dir/braker/prothint
mkdir -p $Work_Dir/braker/relaxed
rm -rf /home/students/q.abbas/Tools/Augustus/config/species/pig

echo "start default of $Species at $(date)"

/home/students/q.abbas/Tools/BRAKER-2.1.6/scripts/braker.pl \
         --species=$Species --softmasking --cores 48 \
         --workingdir=$Work_Dir/braker/default/ \
         --genome=$genome \
         --prot_seq=$proteindb \
         && echo "**********************Default Finished***************************"

/home/students/q.abbas/Tools/BRAKER-2.1.6/scripts/braker.pl \
        --species=pig --softmasking --skipAllTraining --cores 48 \
        --workingdir=$Work_Dir/braker/seeds/ \
        --genome=$genome \
        --hints=$Work_Dir/braker/default/hintsfile.gff \
        && echo "**********************Generative $folder Finished***************************"

/home/students/q.abbas/Tools/ProtHint-2.6.0/bin/prothint.py \
        --threads=48 --workdir $Work_Dir/braker/default/prothint/ $genome $proteindb \
        --geneSeeds=$Work_Dir/braker/default/seeds/augustus.hints.gtf \        && grep -v src=M $Work_Dir/braker/default/prothint/prothint_augustus.gff | grep -v src=C \
        > $Work_Dir/braker/default/prothint/evidence_no_M_C.gff \
        && echo "**********************Prothint  $folder Finished***************************"

/home/students/q.abbas/Tools/BRAKER-2.1.6/scripts/braker.pl \
        --skipAllTraining --species=pig --cores 48 \
        --hints $Work_Dir/braker/default/prothint/evidence_no_M_C.gff \        --genome=$genome \
        --workingdir=$Work_Dir/braker/default/relaxed/ \
        --augustus_args "--minexonintronprob=0 --minmeanexonintronprob=0 \
        --maxtracks=20 --sample=1000 --alternatives-from-sampling=true --temperature=3" \
        --extrinsicCfgFiles /home/students/q.abbas/Tools/BRAKER-2.1.6/scripts/cfg/ep_smallMalus.cfg \
        && echo "**********************Relaxed  $folder Finished***************************"
