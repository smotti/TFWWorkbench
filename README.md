# Description

A mod that modifies the data tables, of the game The Forever Winter, at runtime.

# Why?

Mods that modify the same static assets are most often incompatible with each
other. Meaning the user either has to combine both mods. Which he might not be
capable of. Or he has to pick one mod over the other.

This problem is largely mitigated by modifying these assets, in this case data
tables dynamically, when the game is running. That way new items, weapons, etc
can be added from a set of different mods without conflict. Of course there's
still the risk that multiple mods modify the same item. But resolving that
conflict is made easier by simply editing a json file.

# Installation

1. Download a release: https://github.com/smotti/TFWWorkbench/releases
2. Unpack the release archive
3. Copy the the contents of the archive into a folder named `TFWWorkbench` in UE4SS' `Mods` folder: `Binaries\Win64\ue4ss\Mods`
4. Enable the mod by adding the line `TFWWorkbench : 1` to the `mods.txt` in `Binaries\Win64\ue4ss\Mods\`
5. Create the mod directory `TFWWorkbench` in `Content\Paks\Mods\TFWWorkbench`
6. Either copy the folders from the `Examples` into `Content\Paks\Mods\TFWWorkbench` or create them yourself
  - If you copied the examples remove the json files that they contain

# How to use it

1. Start the game
2. Open the Unreal Engine console by pressing `~` or `F10`
3. Execute the command `DumpDataTables`, this will create a json dump of each supported table in `Content\Paks\Mods\TFWWorkbench`
4. Use the dumped data to write the json files to add/modify/remove entries from the supported data tables (see [Examples](https://github.com/smotti/TFWWorkbench-Lua/tree/main/Examples))

# Supported Data Tables

This is a mapping of the supported data table to the mod folder who's json files
will cause a modifcation of the data table. For example if you create a json file
in the `Item` folder. Than the mod will apply the actions defined in that json file
to the data table `InventoryItemDetailsData`.

- DT_ManufactoringGroups -> CraftingGroups
- DT_ManufactoringRecipies -> CraftingRecipes
- InventoryItemDetailsData -> Item
- Value Data Tables -> ItemValue
- VendorDataTable -> VendorData
- WeaponPartsStatsData -> WeaponPartsStatsData
- WeaponsDetailsData -> WeaponsDetailsData

Note that the mod will automatically add entries to the following two data tables.
As they are required in order for new items to be usable by other parts of the game.

- DT_ItemTags (questionable though)
- DT_TagToRowHandle

The mod will generate the item tag based on its name (row name) and given `ParentTag`.
For example the tag for the `TestItem`, that's added to the game in belows example,
will have the tag `Inventory.Item.TestItem`.

# Schema of "action" json files

```json
[
    {
        "Action": "Add|Modify|Remove",
        "Name": "NameOfTheDataTableRow",
        "Data": {
            ...
        }
    }
]
```

The `Data` follows the same schema as that from the data table dumps.
So you can simply copy a row value from a dump.

# How to define actions

Note that each "action" file can contain multiple different actions.
As they are defined as json objects in a json list.

The actions will be executed in the following order by the mod:
1. Add
2. Modify
3. Remove

The order of the data tabels that are being modified:
1. InventoryItemDetailsData
2. WeaponPartStatsData
3. WeaponsDetailsData
4. Value Data Tables
5. DT_ManufactoringRecipies
6. DT_ManufactoringGroups
7. VendorDataTable

## Action: `Add`

As an example. Create a file named `001_MyItem.json` in `Content\Paks\Mods\TFWWorkbench\Item`.
Open it and add the following contents:
```json
[
    {
        "Action": "Add",
        "Name": "TestItem",
        "Data": {
            "RareLootCategory": "",
            "Category": "Consumable",
            "ExtraTagData": {
                "TagName": "None"
            },
            "ItemLootRadius": 200,
            "ItemIconRadius": 500,
            "ValueRow": {
                "RowName": "TestItem",
                "DataTable": "/Game/Blueprints/Data/Value/LEGACY_ItemValueOverrideData.LEGACY_ItemValueOverrideData"
            },
            "RareLootLocations": "",
            "ItemType": "MedicalSupplies",
            "ItemIcon": "/Game/ArtAssets/UI/Inventory/Textures/ItemPortraits/ItemPortrait_GDC_CerealBox_mre.ItemPortrait_GDC_CerealBox_mre",
            "TacCamHighlight": "Default",
            "MaxStack": 2,
            "ItemMeshTransform": {
                "Translation": {
                    "X": 1,
                    "Y": 0,
                    "Z": 0
                },
                "Scale3D": {
                    "X": 2,
                    "Y": 2,
                    "Z": 2
                },
                "Rotation": {
                    "X": 1,
                    "Y": 0,
                    "Z": 0,
                    "W": 1
                }
            },
            "DropSound": "",
            "ItemSize": {
                "X": 3,
                "Y": 3,
                "Z": 3
            },
            "BattlepointsRowHandle": {
                "RowName": "HealthKit",
                "DataTable": "/Game/BattlePointSystem/DataTables/DT_InventoryItemBattlePoint.DT_InventoryItemBattlePoint"
            },
            "ItemDescription": "Test Description",
            "StartingStack": 1,
            "Weight": 0.6,
            "DropOnDeath": true,
            "LootSound": "",
            "Volume": 8,
            "ItemName": "Test Item",
            "ItemMesh": "/Game/AssetPacksStore/Military_VOL8_Supplies/Meshes/SM_Meal_01a.SM_Meal_01a",
            "ConsumableAbility": "/Game/FW/Player/GameplayAbilities/GA_Player_MedKit.GA_Player_MedKit_C",
            "ExtraDetailsRowName": "None",
            "ItemSubtype": {
                "GameplayTags": [
                    {
                        "TagName": "Item.Healing"
                    },
                    {
                        "TagName": "Item.Consumable"
                    }
                ],
                "ParentTags": [
                    {
                        "TagName": "Item"
                    }
                ]
            }
        }
    }
]
```

When adding an entry to a data table it's better to provide a value for each field.
As a basis you can always use data from the dumped data table json files.

## Action: `Modify`

As an example. If you want to modify the credit value of the "Fist Aid Kit". Create
a file the file `Content\Paks\Mods\TFWWorkbench\ItemValue\001_MyModification.json`.
With the following contents:
```json
[
    {
        "Action": "Modify",
        "Name": "Medical_Heal",
        "Data": {
            "DataTable": "/Game/Blueprints/Data/Value/LEGACY_ItemValueOverrideData.LEGACY_ItemValueOverrideData",
            "Value": 2354,
        }
    }
]
```

Note that, when modifying the value of items, you have to provide the "Value" data
table. As there are multiple data tables that store the value of items.
See below for further things to look out for when modifying entries.

## Action: `Remove`

This example show how to remove the games surplus AK. Create the file
`Content\Paks\Mods\TFWWorkbench\WeaponsDetailsData\001_RemoveSurplus.json`.
Open the file and add the following:
```json
[
    {
        "Action": "Remove",
        "Name": "RFL01A_Surplus"
    }
]
```

Note that when removing an entry you only have to specify the name of the data
table row.

# Best Practices and other Stuff

- When modifying the same entry in a table, especially when modifying complex properties (i.e. lists or maps),
  define multiple `Modify` action entries. For example modifying a vendor's "sold" lists:
  ```json
  [

    {
        "Action": "Modify",
        "Name": "WesternWeaponVendor",
        "Data": {
            "MapedItemsSold": {
                "TestItem": {
                    "VendorLevel": 0,
                    "Quantity": 5,
                    "ItemRowHandle": {
                        "DataTable": "/Game/Blueprints/Data/ItemDetailsData.ItemDetailsData",
                        "RowName": "TestItem"
                    }
                }
            }
        }
    },
    {
        "Action": "Modify",
        "Name": "WesternWeaponVendor",
        "Data": {
            "MapedWeaponsSold": {
                "TestWeapon": {
                    "VendorLevel": 0,
                    "Quantity": 10,
                    "ItemRowHandle": {
                        "DataTable": "/Game/Blueprints/Data/WeaponsDetailsData.WeaponsDetailsData",
                        "RowName": "TestWeapon"
                    }
                }
            }
        }
    }
  ] 
  ```

- The `Modify` action replaces the value of the specified field. This means when modifying lists/maps
  you SHOULD provide the original value(s) as well. For example when modifying a vendor's "sold" lists
  you have to include every item of that list (simply copy it from the json dump file) and make your
  modification(s).
- When adding the value of items via `ItemValue` you need to specify the value data table in which
  to store the item value. This value table is also referenced by the `InventoryItemDetailsData` entry
  of the item. See the [example](https://github.com/smotti/TFWWorkbench-Lua/blob/main/Examples/ItemValue/001_TestItem.json) on how to specify the data table.

# Thanks!

Goes out to:

- Fundog Studios
- trumank
- atenfyr
- FModel developers
- UE4SS developers
  - Special Thanks to: Narknon and Martin
- Special Thanks to the other TFW modders: imi & Meganiikko

Without those people this mod wouldn't exist.