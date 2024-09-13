# Load necessary libraries
library(Maaslin2)
library(tidyverse) # Includes ggplot2, dplyr, tidyr, etc.
library(readr)
library(readxl)

# Set working directory to the project folder
setwd("/Users/...")

# Read in your ASV counts from a delimited text file
Counts <- read_delim("rarefy.table.txt", delim = "\t", col_names = TRUE)

# Transform the data to a tidy format with 'gather' from tidyr package
tidy_counts <- gather(Counts, key = "Sample", value = "count", -Taxon, -ASV)

# Calculate the total counts per sample for normalization
count_sums <- tidy_counts %>%
  group_by(Sample) %>%
  summarise(total = sum(count), .groups = 'drop')

# Join the tidy data frame with the sums and calculate relative abundance
ASV_abundance <- left_join(tidy_counts, count_sums, by = "Sample") %>%
  group_by(ASV, Sample) %>%
  summarise(abundance = count / total, .groups = 'drop')

# Pivot the data to wide format for MaAsLin2 input
GTDB_Abundance <- pivot_wider(ASV_abundance, names_from = ASV, values_from = abundance)

# Save the prepared abundance table
write_csv(GTDB_Abundance, "GTDB_Abundance.csv")

# Read in the metadata, ensuring row names are set for matching
metadata <- read_csv("metadata.csv") %>%
  select(Sample, Practice) %>%
  mutate(row.names = Sample)

# Filter the abundance table to include only samples present in metadata
matched_df <- GTDB_Abundance %>%
  filter(Sample %in% metadata$Sample)

# Run MaAsLin2 with appropriate parameters
results <- Maaslin2(
  input_data = matched_df,
  input_metadata = metadata,
  output = "maaslin_results",
  fixed_effects = c("Practice"),
  max_significance = 0.25
)

# Read in the significant results and merge with taxonomic data
significant_results <- read_excel("maaslin_results/significant_results.xlsx")
taxon_data <- left_join(significant_results, Counts, by = "ASV")

# Generate a basic plot of the results
plot <- ggplot(taxon_data, aes(x = reorder(ASV, coef), y = coef, fill = Practice)) +
  geom_bar(stat = "identity") +
  coord_flip() +
  scale_fill_manual(values = c("#3B1C32", "#6FA37F")) +
  labs(title = "Significant Microbial Associations", x = "ASV", y = "Coefficient")

# Save the plot
ggsave("significant_associations_plot.png", plot = plot, width = 10, height = 8)

# Print the plot to the R console
print(plot)
