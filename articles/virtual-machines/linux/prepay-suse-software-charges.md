---
title: Prepay for software plans - Azure Reservations
description: Learn how you can prepay for software plans to save money over your pay-as-you-go costs.
author: bandersmsft
manager: yashesvi
ms.service: azure-virtual-machines
ms.subservice: billing
ms.custom: linux-related-content
ms.topic: concept-article
ms.date: 10/04/2024
ms.author: banders
# Customer intent: As a financial analyst, I want to understand how to prepay for software plans, so that I can save costs on SUSE and RedHat software usage in Azure and optimize our cloud budget.
---
# Prepay for Azure software plans

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Flexible scale sets 

> [!NOTE]
> The Red Hat Linux Enterprise software reservation plans and renewals are temporarily unavailable due to pending updates to reservation SKUs and pricing. You can disregard any renewal emails until the new plan is available. In the meantime, you can contact your Microsoft or Red Hat Sales Representative to ask about other options until the new plan is available.

When you prepay for your SUSE and RedHat software usage in Azure, you can save money over your pay-as-you-go costs. The discounts only apply to SUSE and RedHat meters and not on the virtual machine usage. You can buy reservations for virtual machines separately for additional savings.

You can buy SUSE and RedHat software plans in the Azure portal. To buy a plan:

- To buy a reservation, you must have owner role or reservation purchaser role on an Azure subscription.
- For Enterprise subscriptions, the **Add Reserved Instances** option must be enabled in the [EA portal](https://ea.azure.com/). If the setting is disabled, you must be an EA Admin for the subscription.
- For the Cloud Solution Provider (CSP) program, the admin agents or sales agents can buy the software plans.

## Buy a software plan

1. Sign in to the Azure portal and go to [Reservations](https://portal.azure.com/#blade/Microsoft_Azure_Reservations/ReservationsBrowseBlade).
2. Click **Add** and then select the software plan that you want to buy.
Fill in the required fields. Any SUSE Linux VM or RedHat VM that matches the attributes of what you buy gets the discount. The actual number of deployments that get the discount depend on the scope and quantity selected.
3. Select a subscription. It's used to pay for the plan.
The subscription payment method is charged the upfront costs for the reservation. The subscription type must be an Enterprise Agreement (offer numbers: MS-AZR-0017P or MS-AZR-0148P) or individual agreement with pay-as-you-go pricing (offer numbers: MS-AZR-0003P or MS-AZR-0023P).
    - For an enterprise subscription, the charges are deducted from the enrollment's Azure Prepayment (previously called monetary commitment) balance or charged as overage.
    - For an individual subscription with pay-as-you-go pricing, the charges are billed to the subscription's credit card or invoice payment method.
4. Select a scope. The scope can cover one subscription or multiple subscriptions (shared scope).
    - Single subscription - The plan discount is applied to matching usage in the subscription.
    - Shared - The plan discount is applied to matching instances in any subscription in your billing context. For enterprise customers, the billing context is the enrollment and includes all subscriptions in the enrollment. For individual plan with pay-as-you-go pricing customers, the billing context is all individual plans with pay-as-you-go pricing subscriptions created by the account administrator.
    - Management group - Applies the reservation discount to the matching resource in the list of subscriptions that are a part of both the management group and billing scope.
    - Single resource group - Applies the reservation discount to the matching resources in the selected resource group only.
5. Select a product to choose the VM size and the image type. The discount applies to the selected VM size only.
6. Select a term. Available term lengths vary by product.
7. Choose a quantity, which is the number of prepaid VM instances that can get the billing discount.
8. Add the product to the cart, review and purchase.

The reservation discount is automatically applied to the software meter that you pre-pay for. VM compute charges aren't covered by the plan. You can purchase the VM reservations separately.

## Discount applies to different SUSE VM sizes

Like reserved VM instances, SUSE Linux plans offer instance size flexibility. Your discount applies even when you deploy a VM that's a different size from the SUSE plan you bought. For more information, see [Understand how the software plan discount is applied](/azure/cost-management-billing/reservations/understand-suse-reservation-charges).

## RedHat plan discount

Plans are available only for Red Hat Enterprise Linux virtual machines. The discount doesn't apply to RedHat Enterprise Linux SAP HANA VMs or RedHat Enterprise Linux SAP Business Apps VMs.

RedHat plan discounts apply only to the VM size that you select at the time of purchase. RHEL plans can't be refunded or exchanged after purchase.


## Self-service cancellation and exchanges not allowed

You can't cancel or exchange a SUSE or RedHat plan that you bought yourself.

Check your usage before purchasing to make sure you buy the right plan. For help to identify what to buy, see [Understand how the software plan discount is applied](/azure/cost-management-billing/reservations/understand-suse-reservation-charges).

## Need help? Contact us.

If you have questions or need help, [create a support request](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest).

## Next steps

To learn how to manage a reservation, see [Manage Azure reservations](/azure/cost-management-billing/reservations/manage-reserved-vm-instance).

To learn more, see the following articles:

- [What are Azure Reservations?](/azure/cost-management-billing/reservations/save-compute-costs-reservations)
- [Manage Reservations in Azure](/azure/cost-management-billing/reservations/manage-reserved-vm-instance)
- [Understand how the SUSE reservation discount is applied](/azure/cost-management-billing/reservations/understand-suse-reservation-charges)
- [Understand reservation usage for your Pay-As-You-Go subscription](/azure/cost-management-billing/reservations/understand-reserved-instance-usage)
- [Understand reservation usage for your Enterprise enrollment](/azure/cost-management-billing/reservations/understand-reserved-instance-usage-ea)
