# Add labels on top of each bar
for bar in bars:
    height = bar.get_height()
    plt.text(
        bar.get_x() + bar.get_width() / 2,  # x position of label
        height,                            # y position of label
        f'{int(height)}',                  # label text
        ha='center',                       # horizontal alignment
        va='bottom'                        # vertical alignment
    )
