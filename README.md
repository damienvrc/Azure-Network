
# Azure CLI
az login
az account set --subscription "YOUR ACCOUNT“
az account show

# New VNET

az network vnet create -g %YOURRESSOURCEGROUP% -n vnet1 --address-prefix 172.16.3.0/24

az network vnet subnet create --address-prefix 172.16.3.16/28 --name sub_01 --resource-group %YOURRESSOURCEGROUP% --vnet-name vnet1
az network vnet subnet create --address-prefix 172.16.3.32/28 --name sub_02 --resource-group %YOURRESSOURCEGROUP% --vnet-name vnet1

# Frond end NSG
az network nsg create --resource-group %YOURRESSOURCEGROUP% --name workshop-frontend-nsg --location northeurope

# Allow HTTP from WAN
az network nsg rule create --resource-group %YOURRESSOURCEGROUP% --nsg-name workshop-frontend-nsg --name Allow-HTTP-All --access Allow --protocol Tcp --direction Inbound --priority 100 --source-address-prefix Internet --source-port-range "*" --destination-address-prefix "*" --destination-port-range 80

# Allow SSH
az network nsg rule create --resource-group %YOURRESSOURCEGROUP% --nsg-name workshop-frontend-nsg --name Allow-SSH-All --access Allow --protocol Tcp --direction Inbound --priority 300 --source-address-prefix MONIP --source-port-range "*" --destination-address-prefix "*" --destination-port-range 22

# Link NSG to front subnet
az network vnet subnet update --vnet-name vnet1 --name sub_01 --resource-group %YOURRESSOURCEGROUP% --network-security-group workshop-frontend-nsg

# Back end NSG
az network nsg create --resource-group %YOURRESSOURCEGROUP% --name workshop-backend-nsg --location northeurope

# Link NSG to back subnet
az network vnet subnet update --vnet-name vnet1 --name sub_02 --resource-group %YOURRESSOURCEGROUP% --network-security-group workshop-backend-nsg

# Allow MSSQL from Front to back
az network nsg rule create --resource-group %YOURRESSOURCEGROUP% --nsg-name workshop-backend-nsg --name Allow-MSSQL-FrontEnd --access Allow --protocol Tcp --direction Inbound --priority 100 --source-address-prefix 172.16.3.0/25 --source-port-range "*" --destination-address-prefix "*" --destination-port-range 1433

# Block Backend outbound
az network nsg rule create --resource-group %YOURRESSOURCEGROUP% --nsg-name workshop-backend-nsg --name Deny-Internet-All --access Deny --protocol Tcp --direction Outbound --priority 300 --source-address-prefix "*" --source-port-range "*" --destination-address-prefix "*" --destination-port-range "*"

# Site to Site VPN (Azure to Azure for exemple purpose)

az network vnet subnet create --vnet-name vnet -n GatewaySubnet -g %YOURRESSOURCEGROUP% --address-prefix 172.16.3.0/29

az network public-ip create -n vnet_01_pip -g %YOURRESSOURCEGROUP% --allocation-method Dynamic

(45min)
az network vnet-gateway create -n vnet_01_gateway -l northeurope --public-ip-address havas_vnet_03_pip -g %YOURRESSOURCEGROUP% --vnet vnet --gateway-type Vpn --sku VpnGw1 --vpn-type RouteBased --no-wait 

## If different ressource group

az network vnet-gateway show -n havas_vnet_03_gateway -g HAVAS-WORKSHOP
Copy ID

az network vpn-connection create –n VNET03toVNET04 -g HAVAS-WORKSHOP --vnet-gateway1 /subscriptions/XXXXXX/resourceGroups/%YOURRESSOURCEGROUP%/providers/Microsoft.Network/virtualNetworkGateways/vnet_01_gateway -l northeurope --shared-key « XXXXXXX" --vnet-gateway2 /subscriptions/XXXXXXX/resourceGroups/%YOURRESSOURCEGROUP%/providers/Microsoft.Network/virtualNetworkGateways/vnet_02_gateway


## If same ressource group

az network vpn-connection create -n VNET03toVNET04 -g %YOURRESSOURCEGROUP% --vnet-gateway1 vnet_01_gateway -l northeurope --shared-key « XXXXXXX" --vnet-gateway2 vnet_02_gateway
az network vpn-connection create -n VNET04toVNET03 -g %YOURRESSOURCEGROUP% --vnet-gateway1 vnet_02_gateway -l northeurope --shared-key « XXXXXXX" --vnet-gateway2 vnet_01_gateway

az network vpn-connection show --name VNET03toVNET04 --resource-group %YOURRESSOURCEGROUP%


