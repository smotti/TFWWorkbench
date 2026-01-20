# Description

A mod that allows for modifying the data tables, of the game The Forever Winter, at runtime.

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
3. Copy the the contents of the archive (excluding `Examples`) into UE4SS' `Mods` folder: `Binaries\Win64\ue4ss\Mods`
4. Enable the mod by adding the line `TFWWorkbench : 1` to the `mods.txt` in `Binaries\Win64\ue4ss\Mods\`
5. Create the mod directory `TFWWorkbench` in `Content\Paks\Mods\TFWWorkbench`
  - Gets also automatically created the first time the game starts with the mod enabled
6. Optionally copy the folders from the `Examples` into `Content\Paks\Mods\TFWWorkbench\DataTables`
  - If you copied the examples remove the json files that they contain
  - Directories will get created automatically the first time the game starts with the mod enabled

# How to use it

1. Start the game
2. Open the Unreal Engine console by pressing `~` or `F10`
3. Execute the command `DumpDataTables`, this will create a json dump of each supported table in `Content\Paks\Mods\TFWWorkbench\Dumps`
4. Use the dumped data to write the json files to add/modify/remove entries from the supported data tables (see [Examples](https://github.com/smotti/TFWWorkbench-Lua/tree/main/Examples))

# Supported Data Tables

This is a mapping of the supported data table to the mod folder whose json files
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
        "Action": "Add|AddTo|ModifyIn|Remove|RemoveFrom|Replace",
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
2. Replace
3. Remove
4. AddTo
5. ModifyIn
6. RemoveFrom

The order of the data tabels that are being modified:
1. InventoryItemDetailsData
2. WeaponPartStatsData
3. WeaponsDetailsData
4. Value Data Tables
5. DT_ManufactoringRecipies
6. DT_ManufactoringGroups
7. VendorDataTable

## Action: `Add`

As an example. Create a file named `001_MyItem.json` in `Content\Paks\Mods\TFWWorkbench\DataTable\Item`.
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

## Action: `AddTo`

This action adds a new element to a property list or map. For example to add a new
weapon to a vendors `MapedWeaponsSold` you can do the following:
```json
[
    {
        "Action": "AddTo",
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
                },
                "RFL20": {
                    "Quantity": 1,
                    "VendorLevel": 1,
                    "ItemRowHandle": {
                        "RowName": "RFL20",
                        "DataTable": "/Game/Blueprints/Data/WeaponsDetailsData.WeaponsDetailsData"
                    }
                }
            }
        }
    }
]
```

This action also supports "property paths". For example to allow Scav Girl to use
the SCAR you can do the following:
```json
[
    {
        "Action": "AddTo",
        "Name": "HRF01",
        "Data": {
            "AllowTags.GameplayTags": [
                {
                    "TagName": "Pawn.Player.Girl"
                }
            ]
        }
    }
]
```

Which will add the tag `Pawn.Player.Girl` to the `GameplayTags` which is a list element
of the `AllowTags` map property.

## Action: `ModifyIn`

The `ModifyIn` action allows for editing of property values of nested maps. For example
if you want to modify the quantity of an item sold by a vendor you can do the following:
```json
[
    {
        "Action": "ModifyIn",
        "Name": "WesternWeaponVendor",
        "Data": {
            "MapedWeaponsSold.TestWeapon": {
                "Quantity": 1
            } 
        }
    }
]
```

Note that this operation supports "property path" as well (`MapedWeaponsSold.TestWeapon`).

## Action: `Replace`

As the name of the action implies. It replaces the value for a property with the
new one specified in the action file.
As an example. If you want to replace the credit value of the "Fist Aid Kit". Create
a file the file `Content\Paks\Mods\TFWWorkbench\DataTable\ItemValue\001_MyModification.json`.
With the following contents:
```json
[
    {
        "Action": "Replace",
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
`Content\Paks\Mods\TFWWorkbench\DataTable\WeaponsDetailsData\001_RemoveSurplus.json`.
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

## Action: `RemoveFrom`

The action `RemoveFrom` allows for removing specific elements for a property whose
value is either a list or map. For example if you want to remove a weapon from a
vendors inventory you can do the following:
```json
[
    {
        "Action": "RemoveFrom",
        "Name": "WesternWeaponVendor",
        "Data": {
            "MapedWeaponsSold": {
                "GRL01": {},
                "SHG01": {}
            }
        }
    }
]
```

This will remove the "S12" shotgun and the "Grenade Launcher" from Grillo. Note that
you have to provide an empty value.

Another example would be to don't allow a character to not be able to use a specific
weapon. For example to disallow the use of the SCAR for Old Man:
```json
[
    {
        "Action": "RemoveFrom",
        "Name": "HRF01",
        "Data": {
            "AllowTags.GameplayTags": [
                {
                    "TagName": "Pawn.Player.OldMan"
                }
            ]
        }
    }
]
```

Note that if you want to remove an element from a list. Which `GameplayTags` is. Than
you have to provide the element that should be removed.

# Limitations

- The actions `AddTo`, `ModifyIn`, and `RemoveFrom` don't work for the following, use
  `Replace` instead:
  - Value data tables
  - WeaponPartStatsData
- The action `ModifyIn` only works on properties whose value is a map, i.e. a vendors
  item lists
  - If you need to modify a value in a list, for example `GameplayTags`, use a combination
    of `RemoveFrom` and `AddTo`

# Guidelines

- When modifying the same entry in a table, especially when replacing complex properties (i.e. lists or maps),
  define multiple `Replace` action entries. For example replacing a vendor's "sold" lists:
  ```json
  [
    {
        "Action": "Replace",
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
        "Action": "Replace",
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

- The `Replace` action replaces the value of the specified field. This means
  when modifying lists/maps you SHOULD provide the original value(s) as well.
  For example when modifying a vendor's "sold" lists you have to include every
  item of that list (simply copy it from the json dump file) and make your
  modification(s).
- When adding the value of items via `ItemValue` you need to specify the value
  data table in which to store the item value. This value table is also referenced
  by the `InventoryItemDetailsData` entry of the item. See the [example](https://github.com/smotti/TFWWorkbench-Lua/blob/main/Examples/ItemValue/001_TestItem.json) on how to specify the data table.
- Use `AddTo`, `ModifyIn`, and `RemoveFrom` where possible! As this improves
  compatibility and reduces the risk of potential conflicts with other mods
  that modify the same data table, data table row, or data table row property.
  For example when adding recipes to a `CraftingGroup` use `AddTo`. Or when
  adding new items to a vendor. Another exmaple is if you want to adjust the
  `VendorLevel` or `Quantity` of an item/weapon sold by a vendor use `ModifyIn`.

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