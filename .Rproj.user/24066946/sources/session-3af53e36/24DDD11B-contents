library(ggplot2)
library(dplyr)

# Define an S4 class named "cell" with slots for identifier, cell type, division rate, and gene expression level.
setClass(
  "cell",
  slots = list(
    identifier = "character",         # Unique cell identifier (e.g., "Cell1")
    cell_type = "factor",             # Cell type (expected: "Stem", "Differentiated", or "Cancer")
    division_rate = "numeric",        # Baseline division rate
    gene_expression_level = "numeric" # Baseline gene expression level
  )
)

# Function to instantiate a single cell.
# Converts the provided cell type into a factor with fixed levels (mapped to numeric labels)
# and creates a new "cell" object.
instantiate_cell <- function(identif, type, division_r, expression_level) {
  actual_cell_type <- factor(
    type,
    levels = c("Stem", "Differentiated", "Cancer"),
    labels = c(1, 2, 3)
  )
  
  current_cell <- new("cell", 
                      identifier = identif, 
                      cell_type = actual_cell_type, 
                      division_rate = division_r, 
                      gene_expression_level = expression_level)
  
  return(current_cell)
}

# Function to create a population of cells.
# It generates 40 cells of each type ("Stem", "Differentiated", "Cancer")
# and stores each cell object in a list.
create_cell_population <- function() {
  cell_types <- rep(c("Stem", "Differentiated", "Cancer"), each = 40)
  current_cells <- vector("list", length(cell_types))
  
  for (index in seq_along(cell_types)) {
    current_cells[[index]] <- instantiate_cell(
      paste0("Cell", index), 
      cell_types[index],
      sample(1:10, size = 1),    # Random division rate between 1 and 10
      sample(1:10, size = 1)     # Random gene expression level between 1 and 10
    )
  }
  
  return(current_cells)
}

# Generate the cell population and store it in "cells".
cells <- create_cell_population()

# Convert the list of cell objects into a flat data frame (Before treatment).
# This data frame extracts each cell’s attributes from the S4 objects.
cell_df_before <- data.frame(
  identifier = sapply(cells, function(cell) cell@identifier),
  cell_type = sapply(cells, function(cell) as.character(cell@cell_type)),
  division_rate = sapply(cells, function(cell) cell@division_rate),
  gene_expression_level = sapply(cells, function(cell) cell@gene_expression_level),
  stringsAsFactors = FALSE
)

# Function to simulate the effect of a growth factor treatment.
# It multiplies gene expression by a fixed amplifier and adjusts the division rate based on cell type.
apply_growth_factor <- function(df) {
  gene_expression_amplifier <- 4  # Multiply gene expression by 4.
  division_rate_changer <- 2        # For "Stem" (1) and "Differentiated" (2), multiply by 2.
  # For "Cancer" (3), divide by 2.
  
  # Update the gene expression level for each cell.
  df$gene_expression_level <- df$gene_expression_level * gene_expression_amplifier
  
  # Update the division rate based on cell type.
  df$division_rate <- ifelse(df$cell_type == 1, 
                             df$division_rate * division_rate_changer,
                             ifelse(df$cell_type == 2, 
                                    df$division_rate * division_rate_changer,
                                    df$division_rate / division_rate_changer))
  
  return(df)
}

# Create the After-treatment data frame by applying the growth factor to the before-treatment data.
cell_df_after <- apply_growth_factor(cell_df_before)

# Optionally, store both before and after data frames together (e.g., in a list)
results <- list(before = cell_df_before, after = cell_df_after)

# Combine the cell data from before and after the experiment
cell_df_before_and_after <- rbind(cell_df_before, cell_df_after)

# Add a new label(Before, After) to help differentiate the cells during the experiment
cell_df_before_and_after$Experiment_Phase <- rep(c("Before", "After"), each = 120)

boxplot_for_group_comparison <- function(df, y_var, y_label = NULL) {
  # If y_label is not provided, use the column name as the label.
  if (is.null(y_label)) {
    y_label <- y_var
  }
  
  # Define a custom labeller to map numeric cell type labels to descriptive names.
  my_labeller <- as_labeller(c("1" = "Stem",
                               "2" = "Differentiated",
                               "3" = "Cancer"))
  
  # Construct the ggplot object using aes_string for dynamic mapping.
  p <- ggplot(df, aes_string(x = "Experiment_Phase", y = y_var, fill = "Experiment_Phase")) +
    geom_boxplot() +
    facet_wrap(~ cell_type, scales = "free", labeller = my_labeller) +
    labs(title = paste(y_label, "Before vs After Treatment"),
         x = "Experiment Phase",
         y = y_label) +
    theme_minimal()
  
  # Print the plot.
  print(p)
}

# Example usage:
# Plot for Division Rate
boxplot_for_group_comparison(cell_df_before_and_after, "division_rate", "Division Rate")

# Plot for Gene Expression Level
#boxplot_for_group_comparison(cell_df_before_and_after, "gene_expression_level", "Gene Expression Level")

make_summary <- function() {
  # Summary statistics for division_rate by Experiment_Phase
  summary_stat_division_rate <- cell_df_before_and_after %>%
    group_by(Experiment_Phase) %>%
    summarise(
      count = n(),                          # Count of cells per phase
      mean_division_rate = mean(division_rate),
      sd_division_rate = sd(division_rate),
      min_division_rate = min(division_rate),
      max_division_rate = max(division_rate),
      .groups = "drop"                      # Ungroup after summarising
    )
  
  # Summary statistics for gene_expression_level by Experiment_Phase
  summary_stat_gene_expression <- cell_df_before_and_after %>%
    group_by(Experiment_Phase) %>%
    summarise(
      count = n(),                          # Count of cells per phase
      mean_gene_expression = mean(gene_expression_level),
      sd_gene_expression = sd(gene_expression_level),
      min_gene_expression = min(gene_expression_level),
      max_gene_expression = max(gene_expression_level),
      .groups = "drop"                      # Ungroup after summarising
    )
  
  # Print the division_rate summary first
  cat("Summary Statistics for Division Rate:\n")
  print(summary_stat_division_rate, n = Inf, width = Inf)
  
  # Print a blank line for clarity between outputs
  cat("\n")
  
  # Then print the gene_expression_level summary
  cat("Summary Statistics for Gene Expression Level:\n")
  print(summary_stat_gene_expression, n = Inf, width = Inf)
}

# Call the function to display the summaries
make_summary()


