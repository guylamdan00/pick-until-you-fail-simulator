#@title PUYF Visualizations { display-mode: "form" }

print(f'\n visualizing data')
# calculate percentage for each level
level_fail_percentages = level_fail_counts.div(level_fail_counts.sum(axis=1), axis=0) * 100

# create the plot
ax = level_fail_percentages.plot(kind='bar', stacked=True, figsize=(12, 6))


print(f'\n Player Distribution per Number of Fails by Level')
#add % labels for values greater than 5%
for rect in ax.patches:
  height = rect.get_height()
  if height > 5:
    ax.text(rect.get_x() + rect.get_width() / 2., rect.get_y() + height / 2.,
            '{:.1f}%'.format(height), ha='center', va='center')

plt.title('Player Distribution per Number of Fails by Level')
plt.xlabel('Level')
plt.ylabel('Percentage of Players')
plt.legend(title='Number of Fails')
plt.show()

# ── % of users whose FINAL fail-count equals X
print('\n % of Users by Final Fails Count')

# Final (max) fails_agg per user
final_fails_per_user = score_data_agg.groupby('uid')['fails_agg'].max().reset_index()

# How many users ended with each fails_agg
users_per_final_fails = (
    final_fails_per_user.groupby('fails_agg')['uid']
    .count()
    .reset_index(name='user_count')
)

# Convert to percent of *all* unique users
total_users = final_fails_per_user['uid'].nunique()
users_per_final_fails['percent_of_users'] = (
    users_per_final_fails['user_count'] / total_users * 100
)

# Plotting
plt.figure(figsize=(12, 6))
ax = sns.barplot(
    data=users_per_final_fails,
    x='fails_agg',
    y='percent_of_users',
    palette='magma'
)

# Add % labels on each bar
for p in ax.patches:
    pct = p.get_height()
    ax.annotate(f'{pct:.1f}%',
                (p.get_x() + p.get_width() / 2, pct),
                ha='center', va='bottom', fontsize=10)

# Axis / grid / title
plt.title('% of Users by Final Fail Count')
plt.xlabel('Final Number of Fails (fails_agg)')
plt.ylabel('% of Users')
plt.xticks(rotation=0)
plt.grid(True, axis='y', linestyle='--', alpha=0.7)
plt.tight_layout()
plt.show()

print(f'\n Actual Fail Probability by Level')
# calculate total number of players
total_players = player_data['uid'].nunique()

# Filter to only fail events
fail_events = score_data_agg[score_data_agg['pick_type'] == 'fail']

# Count unique users who failed at each level
fails_per_level_pct = fail_events.groupby('level')['uid'].nunique().reset_index()
fails_per_level_pct.rename(columns={'uid': 'fail_count'}, inplace=True)


# Calculate percentage
fails_per_level_pct['percent_of_players'] = (fails_per_level_pct['fail_count'] / total_players) * 100

# Plotting
plt.figure(figsize=(12, 6))
ax = sns.barplot(data=fails_per_level_pct, x='level', y='percent_of_players', palette='mako')

# Add labels on top of bars
for p in ax.patches:
    height = p.get_height()
    ax.annotate(f'{height:.1f}%',
                (p.get_x() + p.get_width() / 2, height),
                ha='center', va='bottom',
                fontsize=10)

# Formatting
plt.title('Actual Fail Probability by Level')
plt.xlabel('Level')
plt.ylabel('% of Players')
plt.ylim(0, 100)
plt.grid(True, axis='y', linestyle='--', alpha=0.7)
plt.tight_layout()

# Show plot
plt.show()

#group by level and count unique players with first fail
first_fail_level = score_data_agg[score_data_agg['fail_count'] == 1].groupby('level')['uid'].nunique()


print(f'\n Distribution of Players by Level of First Fail')
plt.figure(figsize=(12, 6))
bars = plt.bar(first_fail_level.index, first_fail_level.values)

# add percentage labels above each bar
for bar in bars:
    height = bar.get_height()
    percentage = (height / total_players) * 100
    plt.text(bar.get_x() + bar.get_width() / 2, height, f'{percentage:.1f}%',
             ha='center', va='bottom', fontsize=10, fontweight='light')

plt.xlabel('Level of First Fail')
plt.ylabel('Number of Players')
plt.title('Distribution of Players by Level of First Fail')

plt.show()

print(f'\n Cumulative Probability of First Fail Until Each Level (Heatmap)')
cumulative_first_fail = first_fail_level.cumsum() / total_players * 100

# convert cumulative probability data to a DataFrame for heatmap
heatmap_data = cumulative_first_fail.to_frame().T  #convert to row format

plt.figure(figsize=(12, 2))
sns.heatmap(heatmap_data, annot=True, fmt=".1f", cmap="Oranges", linewidths=.5, cbar=False)

plt.xlabel('Level')
plt.ylabel('Cumulative Probability (%)')
plt.title("Cumulative Probability of First Fail Until Each Level (Heatmap)")
plt.xticks(rotation=0)
plt.show()

print(f'\n Distribution of Players by Difficulty per Level')
# create heatmap of players by difficulty per level
heatmap_data = score_data_agg.groupby(['level', 'difficulty'])['uid'].nunique().unstack().fillna(0)
plt.figure(figsize=(12, 6))
sns.heatmap(heatmap_data, annot=True, fmt=".0f", cmap="YlGnBu", linewidths=.5)
plt.title("Player Distribution by Difficulty per Level")
plt.xlabel("Difficulty")
plt.ylabel("Level")
plt.show()

print(f'\n Median Cumulative Cost as % of Gems Balance per Fails')
# Create a new column: cost as a % of balance
cost_data['cost_ratio'] = cost_data['cumu_cost'] / cost_data['gems_balance']
cost_data['cost_ratio'] = cost_data['cost_ratio'].replace([np.inf, -np.inf], np.nan).fillna(0)

# Calculate median ratio per fails_agg
median_cost_ratio = cost_data.groupby('fails_agg')['cost_ratio'].median().reset_index()

# Plotting
plt.figure(figsize=(12, 6))
ax = sns.barplot(data=median_cost_ratio, x='fails_agg', y='cost_ratio', palette='coolwarm')

# Add data labels on top of each bar
for p in ax.patches:
    height = p.get_height()
    ax.annotate(f'{height:.2f}',
                (p.get_x() + p.get_width() / 2, height),
                ha='center', va='bottom',
                fontsize=10, fontweight='bold')

# Axis labels and formatting
plt.title('Median Cumulative Cost as % of Gems Balance per Fails')
plt.xlabel('Number of Fails (fails_agg)')
plt.ylabel('Median Cumulative Cost / Gems Balance')
plt.xticks(rotation=0)
plt.grid(True, axis='y', linestyle='--', alpha=0.7)
plt.tight_layout()

# Show plot
plt.show()

print(f'\n % of Players Who Could Afford Continuing by Fails')
# Group by fails_agg and count total and affordable users
afford_stats = cost_data.groupby('fails_agg').agg(
    total_users=('uid', 'nunique'),
    can_afford_users=('can_afford', lambda x: (x == 1).sum())
).reset_index()

# Compute percentage
afford_stats['percent_can_afford'] = (afford_stats['can_afford_users'] / afford_stats['total_users']) * 100

# Plot
plt.figure(figsize=(12, 6))
ax = sns.barplot(data=afford_stats, x='fails_agg', y='percent_can_afford', palette='coolwarm')

# Add data labels
for p in ax.patches:
    height = p.get_height()
    ax.annotate(f'{height:.1f}%',
                (p.get_x() + p.get_width() / 2, height),
                ha='center', va='bottom',
                fontsize=10)

# Axis labels and formatting
plt.title('% of Players Who Could Afford Continuing by Fails')
plt.xlabel('Number of Fails (fails_agg)')
plt.ylabel('% Can Afford')
plt.ylim(0, 105)
plt.xticks(rotation=0)
plt.grid(True, axis='y', linestyle='--', alpha=0.7)
plt.tight_layout()

# Show plot
plt.show()

print(f'\n Cost in Gems to Complete All Levels by Player Percentiles')
# Step 1: Get final cumulative cost per user
final_cost_per_user = cost_data.groupby('uid')['cumu_cost'].max().reset_index()

# Step 2: Compute cost percentiles
percentiles = [0.1, 0.25, 0.5, 0.75, 0.9, 0.95, 0.99]
cost_percentiles = np.percentile(
    final_cost_per_user['cumu_cost'].dropna(),  # <-- drop players with null in max cost (players that didn't need to pay)
    [p * 100 for p in percentiles]
)

print(final_cost_per_user['cumu_cost'].describe())
# Step 3: Prepare for plotting
percentile_df = pd.DataFrame({
    'percentile': percentiles,
    'cumu_cost': cost_percentiles
})

# Step 4: Plotting
plt.figure(figsize=(12, 6))
ax = sns.barplot(data=percentile_df, x='percentile', y='cumu_cost', palette='rocket_r')

# Add labels
for p in ax.patches:
    height = p.get_height()
    ax.annotate(f'{int(height)}',
                (p.get_x() + p.get_width() / 2, height),
                ha='center', va='bottom',
                fontsize=10, fontweight='bold')

# Axis labels and formatting
plt.title('Cost to Complete All Levels by Player Percentiles')
plt.xlabel('Player Percentile')
plt.ylabel('Final Cumulative Gems Cost')
plt.grid(True, axis='y', linestyle='--', alpha=0.7)
plt.tight_layout()

# Show plot
plt.show()

print(f'\n Total Dollar Cost to Complete All Levels by Player Percentiles')

# Step 1: Get final cumulative dollar cost per user (excluding NaNs)
final_dollar_cost_per_user = cost_data.groupby('uid')['cumu_dollar_cost'].max().dropna().reset_index()
print(final_dollar_cost_per_user['cumu_dollar_cost'].describe())
percentiles = [0.1, 0.25, 0.5, 0.75, 0.9, 0.95, 0.99]

# Step 2: Compute percentiles
dollar_cost_percentiles = np.percentile(final_dollar_cost_per_user['cumu_dollar_cost'], [p * 100 for p in percentiles])

# Step 3: Prepare DataFrame for plotting
dollar_percentile_df = pd.DataFrame({
    'percentile': percentiles,
    'cumu_dollar_cost': dollar_cost_percentiles
})

# Step 4: Plot
plt.figure(figsize=(12, 6))
ax = sns.barplot(data=dollar_percentile_df, x='percentile', y='cumu_dollar_cost', palette='crest')

# Add labels
for p in ax.patches:
    height = p.get_height()
    ax.annotate(f'${height:.2f}',
                (p.get_x() + p.get_width() / 2, height),
                ha='center', va='bottom',
                fontsize=10, fontweight='bold')

# Axis labels and formatting
plt.title('Total Dollar Cost to Complete All Levels by Player Percentiles')
plt.xlabel('Player Percentile')
plt.ylabel('Final Cumulative Dollar Cost')
plt.grid(True, axis='y', linestyle='--', alpha=0.7)
plt.tight_layout()

# Show plot
plt.show()

print(f'\n Average Gems Cost per Fail')
# Calculate average gems cost per fail
avg_gems_per_fail = cost_data.groupby('fails_agg')['gems_value'].mean().reset_index()
plt.figure(figsize=(12, 6))
ax = sns.barplot(data=avg_gems_per_fail, x='fails_agg', y='gems_value', palette='coolwarm')

# Add data labels on top of bars
for p in ax.patches:
    height = p.get_height()
    ax.annotate(f'{height:.0f}',
                (p.get_x() + p.get_width() / 2, height),
                ha='center', va='bottom', fontsize=10)

# Customize axes and title
plt.title('Average Gems Cost per Fail')
plt.xlabel('Number of Fails (fails_agg)')
plt.ylabel('Average Gems Cost')
plt.xticks(rotation=0)
plt.grid(True, axis='y', linestyle='--', alpha=0.6)
plt.tight_layout()

# Show the plot
plt.show()

print(f'\n Average Dollar Cost per Fail')

# Calculate average dollar cost per fail
avg_gems_per_fail = cost_data.groupby('fails_agg')['dollar_value'].mean().reset_index()

plt.figure(figsize=(12, 6))
ax = sns.barplot(data=avg_gems_per_fail, x='fails_agg', y='dollar_value', palette='coolwarm')

# Add data labels on top of bars with dollar sign
for p in ax.patches:
    height = p.get_height()
    ax.annotate(f'${height:.2f}',  # <-- Add dollar sign here
                (p.get_x() + p.get_width() / 2, height),
                ha='center', va='bottom', fontsize=10)

# Customize axes and title
plt.title('Average Dollar Cost per Fail')
plt.xlabel('Number of Fails (fails_agg)')
plt.ylabel('Average Dollar Cost')
plt.xticks(rotation=0)
plt.grid(True, axis='y', linestyle='--', alpha=0.6)
plt.tight_layout()

# Show the plot
plt.show()

print(f'\n Average Cost in Gems to Complete All Levels by seg_name')

# Step 1: Get final cost per user
final_cost_per_user = cost_data.groupby('uid')['cumu_cost'].max().reset_index()

# Step 2: Merge with seg_name from player_data
final_cost_per_user = pd.merge(final_cost_per_user, player_data[['uid', 'seg_name']], on='uid', how='left')

# Step 3: Compute average cost per seg_name
avg_cost_per_segment = final_cost_per_user.groupby('seg_name')['cumu_cost'].mean().reset_index()
avg_cost_per_segment.sort_values('cumu_cost', ascending=False, inplace=True)

# Step 4: Plot
plt.figure(figsize=(12, 6))
ax = sns.barplot(data=avg_cost_per_segment, x='seg_name', y='cumu_cost', palette='viridis')

# Add data labels
for p in ax.patches:
    height = p.get_height()
    ax.annotate(f'{int(height)}',
                (p.get_x() + p.get_width() / 2, height),
                ha='center', va='bottom',
                fontsize=10)

# Axis formatting
plt.title('Average Cost to Complete All Levels by Segment')
plt.xlabel('Segment Name')
plt.ylabel('Average Final Cumulative Gems Cost')
plt.xticks(rotation=45)
plt.grid(True, axis='y', linestyle='--', alpha=0.7)
plt.tight_layout()

# Show plot
plt.show()

print(f'\n Average Dollar Cost to Complete All Levels by seg_name')

# Step 1: Get final dollar cost per user
final_dollar_cost_per_user = cost_data.groupby('uid')['cumu_dollar_cost'].max().dropna().reset_index()

# Step 2: Merge with seg_name from player_data
final_dollar_cost_per_user = pd.merge(final_dollar_cost_per_user, player_data[['uid', 'seg_name']], on='uid', how='left')

# Step 3: Compute average dollar cost per segment
avg_dollar_cost_per_segment = final_dollar_cost_per_user.groupby('seg_name')['cumu_dollar_cost'].mean().reset_index()
avg_dollar_cost_per_segment.sort_values('cumu_dollar_cost', ascending=False, inplace=True)

# Step 4: Plot
plt.figure(figsize=(12, 6))
ax = sns.barplot(data=avg_dollar_cost_per_segment, x='seg_name', y='cumu_dollar_cost', palette='viridis')

# Add dollar value labels on top
for p in ax.patches:
    height = p.get_height()
    ax.annotate(f'${height:.2f}',  # formatted with dollar sign
                (p.get_x() + p.get_width() / 2, height),
                ha='center', va='bottom',
                fontsize=10)

# Axis formatting
plt.title('Average Dollar Cost to Complete All Levels by Segment')
plt.xlabel('Segment Name')
plt.ylabel('Average Final Cumulative Dollar Cost')
plt.xticks(rotation=45)
plt.grid(True, axis='y', linestyle='--', alpha=0.7)
plt.tight_layout()

# Show the plot
plt.show()

print(f'\n Total RTP by Percentile')
# Define percentiles
percentiles = [1, 10, 25, 50, 75, 90, 95]

# Compute percentile values and mean
cumu_rtp_percentiles = np.percentile(rtp_filtered['cumu_rtp'].dropna(), percentiles)
mean_rtp = rtp_filtered['cumu_rtp'].mean()

# Create DataFrame
percentile_df = pd.DataFrame({
    'Percentile': [f'{p}%' for p in percentiles] + ['Mean'],
    'Cumulative RTP': list(cumu_rtp_percentiles) + [mean_rtp]
})

# Normalize values for color mapping
norm = Normalize(vmin=percentile_df['Cumulative RTP'].min(), vmax=percentile_df['Cumulative RTP'].max())
cmap = cm.get_cmap('RdYlGn_r')  # Red = low, Green = high
colors = [cmap(norm(value)) for value in percentile_df['Cumulative RTP']]

# Plot
plt.figure(figsize=(12, 6))
ax = sns.barplot(data=percentile_df, x='Percentile', y='Cumulative RTP', palette=colors)

# Annotate bars
for p in ax.patches:
    height = p.get_height()
    ax.annotate(f'{height:.1f}%',
                (p.get_x() + p.get_width() / 2., height),
                ha='center', va='bottom', fontsize=10)

# Formatting
plt.title('Total RTP by Percentile', fontsize=14)
plt.ylabel('RTP at MAX(Leve)')
plt.xlabel('Player Percentile')
plt.grid(True, axis='y', linestyle='--', alpha=0.7)
plt.tight_layout()
plt.show()
