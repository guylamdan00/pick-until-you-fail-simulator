#@title PUYF Main Analysis
import time
from itertools import groupby
import pandas as pd
import numpy as np
import datetime
import random
import openpyxl
import duckdb
import warnings
import matplotlib.pyplot as plt
import seaborn as sns
from matplotlib import cm
from matplotlib.colors import Normalize
import gspread
import pandas as pd
from google.auth import default
from google.colab import auth

warnings.filterwarnings('ignore')

conn = duckdb.connect()
print('reading configuration files')

# read main configuration
auth.authenticate_user()
creds, _ = default()
gc = gspread.authorize(creds)
#insert spreadsheet key below
spreadsheet = gc.open_by_key()

# throw error in case configuration sheets not found in config file
try:
  main_config = spreadsheet.worksheet("main config")
  levels_config = pd.DataFrame(main_config.get())
  cooldown_settings_sheet = spreadsheet.worksheet("cooldown settings")
  cooldown_settings = pd.DataFrame(cooldown_settings_sheet.get())
  gems_slope = spreadsheet.worksheet("gems slope")
  gems_slope = pd.DataFrame(gems_slope.get())
  resource_valuation = spreadsheet.worksheet("resource valuation")
  resource_valuation = pd.DataFrame(resource_valuation.get())
  cost_config = spreadsheet.worksheet("cost config")
  cost_config = pd.DataFrame(cost_config.get())
  segmentation = spreadsheet.worksheet("segmentation")
  segmentation = pd.DataFrame(segmentation.get())
  reward_cost_multipliers = spreadsheet.worksheet("reward cost multipliers")
  reward_cost_multipliers = pd.DataFrame(reward_cost_multipliers.get())
except gspread.exceptions.WorksheetNotFound:
    print("Error: One or both worksheet names are incorrect. Please check the names in Google Sheets.")


# convert first row to header
levels_config.columns = levels_config.iloc[0]
levels_config = levels_config.drop(0)
levels_config.head()
levels_config = levels_config[['level', 'pick_type', 'reward','reward_amount','weight']]

levels_config['weight'] = pd.to_numeric(levels_config['weight'], errors='coerce')
levels_config['sum_weights'] = levels_config.groupby('level')['weight'].transform('sum')
levels_config['probability'] = (levels_config['weight'] / levels_config['sum_weights']) * 100
levels_config['cumu_probability'] = levels_config.groupby('level')['probability'].transform('cumsum')
# Replace empty strings with NaN before converting to int
levels_config['level'] = pd.to_numeric(levels_config['level'], errors='coerce')
levels_config['level'] = np.where(levels_config['level']<1, np.nan, levels_config['level'])
levels_config['level'] = levels_config['level'].dropna(axis=0)  # Remove rows with NaN in 'level' column if necessary
levels_config['level'] = levels_config['level'].astype(int)

# convert first row to header
cooldown_settings.columns = cooldown_settings.iloc[0]
cooldown_settings = cooldown_settings.drop(0)
cooldown_settings = cooldown_settings.dropna(axis=1, thresh=2) #remove columns with 2 or more empty values
cooldown_settings['fail_factor'] = cooldown_settings['fail_factor'].fillna(1)
cooldown_settings.head()
cooldown_settings['fail_factor'] = pd.to_numeric(cooldown_settings['fail_factor'], errors='coerce')
cooldown_settings['difficulty'] = pd.to_numeric(cooldown_settings['difficulty'], errors='coerce')

# convert first row to header
gems_slope.columns = gems_slope.iloc[0]
gems_slope = gems_slope.drop(0)
gems_slope = gems_slope.dropna(axis=1, thresh=2) #remove columns with 2 or more empty values
gems_slope = gems_slope[['dollar value', 'total spins per pp', 'total gems per pp']]
gems_slope['total gems per pp'] = pd.to_numeric(gems_slope['total gems per pp'], errors='coerce')
gems_slope['total gems per pp'] = gems_slope['total gems per pp'].astype(np.float64)
gems_slope['dollar value'] = pd.to_numeric(gems_slope['dollar value'], errors='coerce')
gems_slope.columns = gems_slope.columns.str.strip().str.lower().str.replace(' ', '_')

# convert first row to header
resource_valuation.columns = resource_valuation.iloc[0]
resource_valuation = resource_valuation.drop(0)
resource_valuation = resource_valuation.dropna(axis=1, thresh=2) #remove columns with 2 or more empty values
resource_valuation.columns = resource_valuation.columns.str.lower()
resource_valuation.rename(columns={'spins value': 'spins_value'}, inplace=True)
# ensure column names are stripped of leading/trailing whitespaces
resource_valuation.columns = resource_valuation.columns.str.strip()

# extract the spins value for Gems reward
gems_row = resource_valuation.loc[resource_valuation['reward'].str.strip() == 'Gems']

# make sure 'spins' is treated as a numeric value
gems_row['spins'] = pd.to_numeric(gems_row['spins_value'], errors='coerce')

# get the single value (assuming only one row for 'Gems')
gems_to_spins = gems_row['spins'].values[0] if not gems_row.empty else None
resource_valuation['spins_value'] = pd.to_numeric(resource_valuation['spins_value'], errors='coerce')

resource_valuation['gems_value'] = np.where(resource_valuation['reward']=='Gems', 1, resource_valuation['spins_value'] / gems_to_spins)
resource_valuation['gems_value'] = np.where(resource_valuation['gems_value']<10, np.round(resource_valuation['gems_value'], 1),  np.ceil(resource_valuation['gems_value']))
resource_valuation.drop_duplicates(subset='reward', inplace=True)

# create no fails config to add rewards for users in case of fail
levels_config_no_fail = levels_config[levels_config['pick_type']!='fail']
levels_config_no_fail['weight'] = pd.to_numeric(levels_config_no_fail['weight'], errors='coerce')
levels_config_no_fail['sum_weights'] = levels_config_no_fail.groupby('level')['weight'].transform('sum')
levels_config_no_fail['probability'] = (levels_config_no_fail['weight'] / levels_config_no_fail['sum_weights'])
levels_config_no_fail['prob_to'] = levels_config_no_fail.groupby('level')['probability'].transform('cumsum')
levels_config_no_fail['prob_from'] = levels_config_no_fail.groupby('level')['prob_to'].shift(1)
levels_config_no_fail['prob_from'] = levels_config_no_fail['prob_from'].fillna(0)
levels_config_no_fail = pd.merge(levels_config_no_fail, resource_valuation[['reward', 'gems_value']], on='reward', how='left')
levels_config_no_fail['reward_amount'] = pd.to_numeric(levels_config_no_fail['reward_amount'], errors='coerce')
levels_config_no_fail['gems_value'] = levels_config_no_fail['gems_value'].fillna(0)
levels_config_no_fail.rename(columns={'reward':'reward_after_fail', 'reward_amount':'reward_amount_after_fail', 'gems_value':'gems_value_after_fail'}, inplace=True)
levels_config_no_fail = levels_config_no_fail[['level', 'prob_from', 'prob_to', 'reward_after_fail', 'reward_amount_after_fail', 'gems_value_after_fail']]

# convert first row to header
cost_config.columns = cost_config.iloc[0]
cost_config = cost_config.drop(0)
cost_config = cost_config.dropna(axis=1, thresh=2) #remove columns with 2 or more empty values
cost_config = cost_config[['level', 'level_cost_factor']]
cost_config['level'] = pd.to_numeric(cost_config['level'], errors='coerce')
cost_config['level_cost_factor'] = pd.to_numeric(cost_config['level_cost_factor'], errors='coerce')

# convert first row to header
segmentation.columns = segmentation.iloc[0]
segmentation = segmentation.drop(0)
segmentation = segmentation[['seg_name', 'balance_from', 'balance_to', 'dslp_from', 'dslp_to']]
segmentation['balance_from'] = pd.to_numeric(segmentation['balance_from'], errors='coerce')
segmentation['balance_to'] = pd.to_numeric(segmentation['balance_to'], errors='coerce')
segmentation['dslp_from'] = pd.to_numeric(segmentation['dslp_from'], errors='coerce')
segmentation['dslp_to'] = pd.to_numeric(segmentation['dslp_to'], errors='coerce')

# convert first row to header
reward_cost_multipliers.columns = reward_cost_multipliers.iloc[0]
reward_cost_multipliers = reward_cost_multipliers.drop(0)
reward_cost_multipliers = reward_cost_multipliers[['seg_name', 'level_cost_multiplier', 'energy_reward_factor', 'gems_reward_factor', 'coinssi_reward_factor']]
reward_cost_multipliers['level_cost_multiplier'] = pd.to_numeric(reward_cost_multipliers['level_cost_multiplier'], errors='coerce')
reward_cost_multipliers['energy_reward_factor'] = pd.to_numeric(reward_cost_multipliers['energy_reward_factor'], errors='coerce')
reward_cost_multipliers['gems_reward_factor'] = pd.to_numeric(reward_cost_multipliers['gems_reward_factor'], errors='coerce')
reward_cost_multipliers['coinssi_reward_factor'] = pd.to_numeric(reward_cost_multipliers['coinssi_reward_factor'], errors='coerce')

print('generating players')
# generate player data
player_data = pd.DataFrame({'uid': list(range(1, 100001))})
player_data.sort_values(['uid'], inplace=True)
player_data['difficulty'] = 0
player_data['fail_count'] = 0
player_data['rand'] = [random.random() for _ in player_data['uid']]

# assign gems balance according to percentile
def assign_random_gems(rand):
    if rand <= 0.05:
        return 0
    elif rand <= 0.1:
        return np.random.uniform(0, 15)
    elif rand <= 0.25:
        return np.random.uniform(15, 65)
    elif rand <= 0.5:
        return np.random.uniform(65, 150)
    elif rand <= 0.75:
        return np.random.uniform(150, 260)
    elif rand <= 0.9:
        return np.random.uniform(260, 400)
    elif rand <= 0.95:
        return np.random.uniform(400, 1000)
    else:
        return np.random.uniform(1000, 2000)

player_data['gems_balance'] = player_data['rand'].apply(assign_random_gems).astype(int)

percentiles = [0.1, 0.25, 0.5, 0.75, 0.9, 0.95, 0.99]

# Get gems_balance values at these percentiles
percentile_values = player_data['gems_balance'].quantile(percentiles)

print("Gems Balance at Selected Percentiles:")
print(percentile_values)

# === Assign segment based on segmentation rules ===
# Start by assigning default segment
player_data['seg_name'] = 'unsegmented'

# Iterate through each segmentation rule
for _, seg in segmentation.iterrows():
    condition = pd.Series([True] * len(player_data))

    if 'balance_from' in seg and 'gems_balance' in player_data.columns:
        condition &= player_data['gems_balance'] >= seg['balance_from']
    if 'balance_to' in seg and 'gems_balance' in player_data.columns:
        condition &= player_data['gems_balance'] < seg['balance_to']
    if 'dslp_from' in seg and 'dslp' in player_data.columns:
        condition &= player_data['dslp'] >= seg['dslp_from']
    if 'dslp_to' in seg and 'dslp' in player_data.columns:
        condition &= player_data['dslp'] < seg['dslp_to']

    # Only update seg_name where it hasn't been assigned yet and the condition is met
    player_data.loc[(player_data['seg_name'] == 'unsegmented') & condition, 'seg_name'] = seg['seg_name']

print(f'number of unsegmented users: {len(player_data[player_data["seg_name"] == "unsegmented"])}')

level = 1

score_data = pd.DataFrame()
duplicate_columns = ['prob_from', 'prob_to', 'fail_factor', 'level', 'pick_type', 'item_id', 'reward', 'reward_amount', 'weight', 'sum_weights', 'probability', 'cumu_probability', 'difficulty_1','level_1','reward_after_fail','reward_amount_after_fail','gems_value_after_fail']

print('calculating picks')
start_time = time.time()
for level in range(1, max(levels_config['level'] + 1)):
    # level +=1
    player_data.drop(columns=duplicate_columns, errors='ignore', inplace=True)
    player_data['pick_result'] = [random.random() for _ in player_data['uid']]
    # make sure players don't receive 0 to avoid getting fail when fail factor is 0
    player_data['pick_result'] = np.where(player_data['pick_result']==0, 1,player_data['pick_result'])
    player_data.sort_values(['difficulty', 'pick_result'], inplace=True)


    # filter current level
    current_level = levels_config[levels_config['level']==level]
    current_level['item_id'] = current_level.groupby('level').cumcount() + 1

    # join cooldown on each row
    current_level = pd.merge(current_level, cooldown_settings, how='cross')
    current_level['weight'] = pd.to_numeric(current_level['weight'], errors='coerce')
    current_level['difficulty'] = current_level['difficulty'].astype(int)
    current_level.sort_values(['difficulty'], inplace=True)
    current_level['weight'] = np.where(current_level['pick_type']=='fail', current_level['weight'] * current_level['fail_factor'],current_level['weight'])
    current_level['sum_weights'] = current_level.groupby('difficulty')['weight'].transform('sum')
    current_level['probability'] = (current_level['weight'] / current_level['sum_weights'])
    current_level.sort_values(['difficulty','probability'], inplace=True)
    current_level['prob_to'] = current_level.groupby('difficulty')['probability'].transform('cumsum')
    current_level = current_level.sort_values(['difficulty','prob_to'])
    current_level['prob_from'] = current_level.groupby('difficulty')['prob_to'].shift(1)
    current_level['prob_from'] = current_level['prob_from'].fillna(0)
    current_level = current_level[['difficulty', 'prob_from', 'prob_to', 'fail_factor', 'level', 'pick_type', 'item_id', 'reward', 'reward_amount', 'weight', 'sum_weights', 'probability', 'cumu_probability']]

    # Get no-fail config for this level
    no_fail_config = levels_config_no_fail[levels_config_no_fail['level'] == level]
    no_fail_config['prob_from'] = pd.to_numeric(no_fail_config['prob_from'], errors='coerce')
    no_fail_config['prob_to'] = pd.to_numeric(no_fail_config['prob_to'], errors='coerce')

    # assign pick to player according to difficulty (cooldown setting)
    player_data = conn.execute(f'''
    select *
    from player_data as p
    left join current_level as c
    on p.difficulty = c.difficulty
    and p.pick_result > c.prob_from
    and p.pick_result <= c.prob_to
    ''').df()

    conn.register("no_fail_config", no_fail_config)
    player_data = conn.execute(f'''
    select p.*,
    nf.reward_after_fail,
    nf.reward_amount_after_fail,
    nf.gems_value_after_fail
    from player_data as p
    left join no_fail_config as nf
    on p.level = nf.level
    and p.pick_result > nf.prob_from
    and p.pick_result <= nf.prob_to
    ''').df()

    # update fails count
    player_data['fail_count'] = np.where(player_data['pick_type']=='fail', 1, 0)
    score_data = pd.concat([score_data, player_data])
    player_data['difficulty'] = np.where(player_data['pick_type']=='fail', min(cooldown_settings['difficulty']), player_data['difficulty'] + 1)
    player_data['difficulty'] = np.where(player_data['difficulty']>max(cooldown_settings['difficulty']), max(cooldown_settings['difficulty']), player_data['difficulty'])
    # player_data = player_data[player_data['uid']<50] # delete this row for QA only!
    # print(len(player_data))
    # score_data = score_data[score_data['uid']<50] # delete this row for QA only!
    # print(len(score_data))
    print(f'pick number: {level}, Acc. calculation time:{time.time() - start_time}')

print('aggragating data')
score_data.drop(columns='difficulty_1', errors='ignore', inplace=True)
score_data_agg = score_data[['uid', 'seg_name', 'level', 'difficulty', 'pick_result', 'pick_type', 'fail_count',  'reward', 'reward_amount', 'item_id','reward_after_fail','reward_amount_after_fail','gems_value_after_fail']]
score_data_agg.sort_values(['uid', 'level'], inplace=True)
score_data_agg = pd.merge(score_data_agg, reward_cost_multipliers, on='seg_name', how='left')
score_data_agg['fail_count'] = score_data_agg.groupby('uid')['fail_count'].transform('cumsum')
score_data_agg['fail_count'] = np.where(score_data_agg['pick_type']=='fail', score_data_agg['fail_count'], 0)
score_data_agg['fails_agg'] = np.where(score_data_agg['pick_type'] == 'fail', score_data_agg['fail_count'], np.nan)
score_data_agg['fails_agg'] = score_data_agg.groupby('uid')['fails_agg'].ffill()
score_data_agg['fails_agg'].fillna(0, inplace=True)
score_data_agg['fails_agg'] = score_data_agg['fails_agg'].astype(int)
score_data_agg['reward_amount'] = score_data_agg['reward_amount'].fillna(0)
score_data_agg['reward_amount'] = pd.to_numeric(score_data_agg['reward_amount'], errors='coerce')

# Ensure reward is lowercase and stripped
score_data_agg['reward_clean'] = score_data_agg['reward'].str.lower().str.strip()

# Apply reward factor based on reward type
score_data_agg['reward_amount'] = np.where(
    score_data_agg['reward_clean'] == 'gems',
    score_data_agg['reward_amount'] * score_data_agg['gems_reward_factor'],
    np.where(
        score_data_agg['reward_clean'] == 'energy',
        score_data_agg['reward_amount'] * score_data_agg['energy_reward_factor'],
        np.where(
            score_data_agg['reward_clean'] == 'coinssi',
            score_data_agg['reward_amount'] * score_data_agg['coinssi_reward_factor'],
            score_data_agg['reward_amount']  # leave unchanged if reward type is unknown
        )
    )
)

# Round reward amount after multiplying by factors
score_data_agg['reward_amount'] = np.where(
    (score_data_agg['reward_clean'] == 'gems') | (score_data_agg['reward_clean'] == 'energy'),
    round(score_data_agg['reward_amount']),
    score_data_agg['reward_amount']
)

# Get rewards valuation
score_data_agg = pd.merge(score_data_agg, resource_valuation[['reward', 'gems_value']], on='reward', how='left')
score_data_agg['gems_value'] = np.ceil(score_data_agg['gems_value'] * score_data_agg['reward_amount'] * score_data_agg['level_cost_multiplier'])
score_data_agg['gems_value'] = score_data_agg['gems_value'].fillna(0)
score_data_agg = score_data_agg.sort_values(['uid', 'level', 'gems_value_after_fail'], ascending=[True, True, False])
score_data_agg['gems_value'] = np.where(score_data_agg['pick_type']=='fail', score_data_agg['gems_value_after_fail'], score_data_agg['gems_value'])

# Apply rewards after fail according to pick type
score_data_agg['reward_after_fail'] = np.where(score_data_agg['pick_type']=='prize', np.nan, score_data_agg['reward_after_fail'])
score_data_agg['reward_amount_after_fail'] = np.where(score_data_agg['pick_type']=='prize', np.nan, score_data_agg['reward_amount_after_fail'])

# Ensure reward is lowercase and stripped
score_data_agg['reward_after_fail_clean'] = score_data_agg['reward_after_fail'].str.lower().str.strip()

# Apply reward factor based on reward type
score_data_agg['reward_amount_after_fail'] = np.where(
    score_data_agg['reward_after_fail_clean'] == 'gems',
    score_data_agg['reward_amount_after_fail'] * score_data_agg['gems_reward_factor'],
    np.where(
        score_data_agg['reward_after_fail_clean'] == 'energy',
        score_data_agg['reward_amount_after_fail'] * score_data_agg['energy_reward_factor'],
        np.where(
            score_data_agg['reward_after_fail_clean'] == 'coinssi',
            score_data_agg['reward_amount_after_fail'] * score_data_agg['coinssi_reward_factor'],
            score_data_agg['reward_amount_after_fail']  # leave unchanged if reward type is unknown
        )
    )
)

# Round reward amount after multiplying by factors
score_data_agg['reward_amount_after_fail'] = np.where(
    (score_data_agg['reward_after_fail_clean'] == 'gems') | (score_data_agg['reward_after_fail_clean'] == 'energy'),
    round(score_data_agg['reward_amount_after_fail']),
    score_data_agg['reward_amount_after_fail']
)

# Multiply gems after fail values by reward amount and level cost multiplier according to segment
score_data_agg['gems_value_after_fail'] = np.ceil(score_data_agg['gems_value_after_fail'] * score_data_agg['reward_amount_after_fail'] * score_data_agg['level_cost_multiplier'])
score_data_agg['gems_value_after_fail'] = np.where(score_data_agg['pick_type']=='prize', np.nan, score_data_agg['gems_value_after_fail'])
score_data_agg['gems_value'] = np.where(score_data_agg['pick_type']=='fail', score_data_agg['gems_value_after_fail'], score_data_agg['gems_value'])


level_fails = score_data_agg[score_data_agg['pick_type']=='fail']

# cost analysis per fail
cost_data = score_data_agg.groupby(['uid', 'fails_agg'])['gems_value'].sum().reset_index()
cost_data = pd.merge(cost_data, player_data[['uid', 'gems_balance']], on='uid', how='left')
cost_data['fails_agg'] +=1
cost_data = pd.merge(cost_data, level_fails[['uid','level','fails_agg']], on=['uid','fails_agg'], how='left')
cost_data = cost_data.dropna(subset=['level'])
cost_data['level'] = cost_data['level'].astype(int)
cost_data = pd.merge(cost_data, cost_config[['level', 'level_cost_factor']], on='level', how='left')
cost_data.rename(columns={'level':'level_of_fail'}, inplace=True)
cost_data['gems_value'] = np.ceil(cost_data['gems_value'] * cost_data['level_cost_factor'])
cost_data['cumu_cost'] = cost_data.groupby('uid')['gems_value'].transform('cumsum')
cost_data['balance_after'] = cost_data['gems_balance'] - cost_data['cumu_cost']
cost_data['balance_after'] = np.where(cost_data['balance_after'] < 0, 0, cost_data['balance_after'])
cost_data['can_afford'] = np.where(cost_data['gems_balance'] < cost_data['cumu_cost'], 0, 1)
cost_data['gems_value'] = pd.to_numeric(cost_data['gems_value'], errors='coerce')
cost_data = pd.merge_asof(
    cost_data.sort_values('gems_value'),
    gems_slope[['dollar_value', 'total_gems_per_pp']],
    left_on='gems_value',
    right_on='total_gems_per_pp',
    direction='forward'  # find the nearest larger or equal value
)
cost_data = cost_data.sort_values(['uid', 'fails_agg'], ascending=True)
cost_data['dollar_value'] = np.where(cost_data['can_afford']==1, np.nan, cost_data['dollar_value'])
cost_data['total_gems_per_pp'] = np.where(cost_data['can_afford']==1, np.nan, cost_data['total_gems_per_pp'])
cost_data['cumu_dollar_cost'] = cost_data.groupby('uid')['dollar_value'].transform('cumsum')

# calculate rtp per level
rtp_data = pd.merge(score_data_agg[['uid', 'level', 'gems_value']], cost_data[['uid', 'level_of_fail', 'cumu_cost']], left_on=['uid', 'level'], right_on= ['uid', 'level_of_fail'], how='left')
rtp_data['cumu_gems_value'] = rtp_data.groupby('uid')['gems_value'].transform('cumsum')
rtp_data['cumu_cost'] = rtp_data.groupby('uid', group_keys=False)['cumu_cost'].transform('ffill')
rtp_data['cumu_cost'].fillna(0, inplace=True)
rtp_data['cumu_rtp'] = np.where(rtp_data['cumu_cost']==0, np.nan, np.ceil(rtp_data['cumu_gems_value'] / rtp_data['cumu_cost'] * 100))

rtp_filtered = rtp_data[rtp_data['level']==max(levels_config['level'])]

# Aggregate fails by level to check the fail %
levels_agg = pd.DataFrame({'level': list(range(0, 16))})
levels_agg['fails_per_level'] = score_data_agg[score_data_agg['fail_count'] == 1].groupby('level')['pick_type'].count()
levels_agg['fails_per_level'].fillna(0, inplace=True)
levels_agg['level_fail_rate'] = levels_agg['fails_per_level'] / len(pd.unique(score_data_agg['uid'])) * 100

# calculate fail counts
level_fail_counts = score_data_agg.groupby(['level', 'fails_agg'])['uid'].nunique().unstack().fillna(0)

print(f'\n Analysis complete')
