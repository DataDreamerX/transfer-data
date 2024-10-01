# Manually create legend handles for the solid and dashed line styles
handles_latency = [mlines.Line2D([], [], color=color, marker='o', linestyle='-', label=f'{category} (Latency)')
                   for category, color in zip(resampled_pdf['category'].unique(), green_palette)]

handles_result_count = [mlines.Line2D([], [], color=color, marker='x', linestyle='--', label=f'{category} (Result Count)')
                        for category, color in zip(resampled_pdf['category'].unique(), green_palette)]

# Combine the latency and result count legend handles
handles = handles_latency + handles_result_count

# Add the custom legend
ax1.legend(handles=handles, title="Categories")
