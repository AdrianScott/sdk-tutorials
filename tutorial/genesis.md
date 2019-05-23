###Genesis

The AppModule interface includes a number of functions for use in initializing and exporting GenesisState for the chain. The `ModuleBasicManager` calls these functions on each module when starting, stopping or exporting the chain. Here is a very basic implementation that you can expand upon.

Lets start by adding the `GenesisState` struct in `x/nameservice/types`. This struct is intended to hold all the application state for our module. As our nameservice is still quite simple, we need only store the Whois data.

```go
type GenesisState struct {
	WhoisRecords []Whois `json:"whois_records"`
}
```

Next `x/nameservice/genesis.go` and add the following code:

```go

package nameservice

import (
	"fmt"
	sdk "github.com/cosmos/cosmos-sdk/types"
	abci "github.com/tendermint/tendermint/abci/types"
)

func ValidateGenesis(data GenesisState) error {

	for _, record := range data.WhoisRecords {
		if record.Owner == nil {
			return fmt.Errorf("Invalid WhoisRecord: Value: %s. Error: Missing Owner", record.Value)
		}
		if record.Value == "" {
			return fmt.Errorf("Invalid WhoisRecord: Owner: %s. Error: Missing Value", record.Owner)
		}
		if record.Price == nil {
			return fmt.Errorf("Invalid WhoisRecord: Value: %s. Error: Missing Price", record.Value)
		}
	}
	return nil
}

func DefaultGenesisState() GenesisState {
	return GenesisState{
		WhoisRecords: []Whois{},
		Params:       DefaultParams(),
	}
}

func InitGenesis(ctx sdk.Context, keeper Keeper, data GenesisState) []abci.ValidatorUpdate {

	for _, record := range data.WhoisRecords {
		keeper.SetWhois(ctx, record.Value, record)
	}
	return []abci.ValidatorUpdate{}
}

func ExportGenesis(ctx sdk.Context, keeper Keeper) GenesisState {

 var records []Whois
 iterator := keeper.GetNamesIterator(ctx)

 for ; iterator.Valid(); iterator.Next() {
   name := string(iterator.Key())
   var whois Whois
   whois := keeper.cdc.MusUnMarshalBinaryBare(iterator.Value(), &whois)
   records = append(records, whois)
 }
 return GenesisState{WhoisRecords: records}

}
```

A few notes about the above code:

- `ValidateGenesis()` validates the provided genesis state to ensure that expected invariants hold
- `DefaultGenesisState()` used mostly for testing. This provides a minimal GenesisState.
- `InitGenesis()` called on chain start, this function imports genesis state into the keeper.
- `ExportGenesis()` called after stopping the chain, this function loads application state into a GenesisState stuct to later be exported to `genesis.json` alongside data from the other modules.

### Now your module has everything it needs to be [incorporated into your Cosmos SDK application](./app-complete.md)!