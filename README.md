```gql
# account entity can be used to get info about current account state and absorb account actions 
type Account @entity {
  id: ID!
  
  transfers: [AccountTransfer!] @derivedFrom(field: "account")
  
  contributions: [Contribution!] @derivedFrom(field: "account")
  crowdloans: [Contributor!] @derivedFrom(field: "account") # crowdloans there account get participation
  
  stakingInfo: StakingInfo @derivedFrom(field: "stash")
  activeBond: BigInt! # current bond balance
  totalReward: BigInt!
  totalSlash: BigInt!
  rewards: [Reward!] @derivedFrom(field: "account")
  slashes: [Slash!] @derivedFrom(field: "account")
  bonds: [Bond!] @derivedFrom(field: "account")
  validatorHistory: [EraValidator!] @derivedFrom(field: "stash") # validator history for each era
  nominatorHistory: [EraNominator!] @derivedFrom(field: "stash") # nominator history for each era

  lastUpdateBlock: BigInt!
}

enum StakingRole {
  Validator
  Nominator
  Idle
}

enum PayeeType {
  Staked
  Stash
  Controller
  Account
  None
}

# current information about stash, controller, payee and staking role.
# See https://docs.substrate.io/rustdocs/latest/pallet_staking/index.html
type StakingInfo @entity {
  id: ID! #stash Id
  stash: Account! @unique
  controller: Account!
  payee: Account
  payeeType: PayeeType!
  role: StakingRole!
  commission: Int
}

# information about era, validaotors and nominators 
type Era @entity {
  id: ID!
  index: Int!
  timestamp: DateTime!
  startedAt: Int!
  endedAt: Int
  total: BigInt!
  validatorsCount: Int!
  nominatorsCount: Int!
  validators: [EraValidator] @derivedFrom(field: "era")
  nominators: [EraNominator] @derivedFrom(field: "era")
}

type EraStakingPair @entity {
  id: ID! #era + validatorId + nominatorId
  era: Era!
  nominator: EraNominator
  validator: EraValidator
  vote: BigInt!
}

# information abount validator in era: self/total bond, nominators and their votes
type EraValidator @entity {
  id: ID! #era + stashId
  stash: Account!
  era: Era!
  selfBonded: BigInt!
  totalBonded: BigInt!
  commission: Int
  nominators: [EraStakingPair] @derivedFrom(field: "validator")
}

# information abount nominator in era: bond, validators and votes for them in each era
type EraNominator @entity {
  id: ID! #era + stashId
  stash: Account!
  era: Era!
  bonded: BigInt!
  validators: [EraStakingPair] @derivedFrom(field: "nominator")
}

# information about known parachains and their crowdloans
type Parachain @entity {
  id: ID! #paraId
  name: String
  paraId: Int
  crowdloans: [Crowdloan!] @derivedFrom(field: "parachain")
  relayChain: String
}

enum CrowdloanStatus {
  CREATED
  WON
  DISSOLVED
}

enum TransferDicrection {
  FROM
  TO
}

type Contributor @entity {
  id: ID!
  crowdloan: Crowdloan!
  account: Account!
  amount: BigInt!
}

# information about known crowdloans.
# See https://wiki.polkadot.network/docs/learn-crowdloans
type Crowdloan @entity {
  id: ID!
  cap: BigInt!
  firstPeriod: BigInt!
  lastPeriod: BigInt!
  end: BigInt!
  contributors: [Contributor!] @derivedFrom(field: "crowdloan")
  raised: BigInt!
  parachain: Parachain
  blockNumber: BigInt @index
  createdAt: DateTime
}

interface Item {
  timestamp: DateTime
  blockNumber: BigInt
  extrinsicHash: String
  amount: BigInt
}

interface HasTotal {
  total: BigInt
}

interface HasEra {
  era: Int
}

interface CanFail {
  success: Boolean
}

type Contribution implements Item & CanFail @entity {
  id: ID!
  timestamp: DateTime
  blockNumber: BigInt @index
  extrinsicHash: String @index
  crowdloan: Crowdloan
  success: Boolean @index
  account: Account!
  amount: BigInt
}

type Transfer implements Item & CanFail @entity {
  id: ID!
  timestamp: DateTime
  blockNumber: BigInt @index
  extrinsicHash: String @index
  to: Account!
  from: Account!
  amount: BigInt
  success: Boolean @index
}

# entity for linking account and transfer
type AccountTransfer @entity {
  id: ID!
  transfer: Transfer
  account: Account!
  direction: TransferDicrection
}

type Reward implements Item & HasTotal & HasEra @entity  {
  id: ID!
  timestamp: DateTime
  blockNumber: BigInt @index
  extrinsicHash: String @index
  account: Account!
  amount: BigInt
  era: Int
  validator: String
  total: BigInt
}

type Slash implements Item & HasTotal & HasEra @entity  {
  id: ID!
  timestamp: DateTime
  blockNumber: BigInt @index
  extrinsicHash: String @index
  account: Account!
  amount: BigInt
  era: Int
  total: BigInt
}

enum BondType {
  Bond
  Unbond
}

type Bond implements Item & HasTotal & CanFail @entity  {
  id: ID!
  timestamp: DateTime
  blockNumber: BigInt @index
  extrinsicHash: String @index
  account: Account!
  amount: BigInt
  total: BigInt
  success: Boolean @index
  type: BondType
}
