Approach Note:

1. Churn settlement api should work with the transactor block as its mandatory to set initiator as transactor.\
3. Post approval from the admin money movement will be done from IND10 to IND04/IND01
4. Irrespective of currency factor, the ammount will be settled to respective accounts.
5. Its a maker and checker process.


Flow Diagram:

https://comviva.atlassian.net/wiki/download/thumbnails/38481820/image2024-5-21_12-28-33.png?version=1&modificationDate=1716272821321&cacheVersion=1&api=v2&width=1272&height=400


Admin will intiate the churn settlement ammount from system wallet (IND10) to reapective User(Business.Subscriber) or IND04

               |=> Subscriber/Business =>|
               |            user         | 
Admin Request--|                         |=> Checker settlement process  => Approval Stage => Settled the churn amount => Db update is
               |                         |   goes to checker for confirmtion.                 received from request.      settled
			   |=> IND04               =>|
			   
			   
			   
			   


SFM Flow:


<serviceFlow id="CHURN_SETTLEMENT" name="CHURN_SETTLEMENT" enabled="true" serviceCode="CHURN_SETTLEMENT">
    <description>Churn Settlement</description>
    <flowMode>NONE</flowMode>
    <steps>
        <step id="serviceRequest.validation" name="serviceRequest.validation" enabled="true" removeSensitiveFieldsAfter="false"/>
        <step id="party.validation" name="party.validation.v2" enabled="true" removeSensitiveFieldsAfter="false"/>
        <step id="validate.initiator" name="validate.initiator.v2" enabled="true"/>
        <step id="churn.settlement" name="churn.settlement" enabled="true" removeSensitiveFieldsAfter="false"/>
        <step id="charges.calculation" name="charges.calculation" enabled="true" removeSensitiveFieldsAfter="false"/>
        <step id="account.validation" name="account.validation.v2" enabled="true" removeSensitiveFieldsAfter="false"/>
        <step id="authorize.user" name="authorize.user" privilege-code="CHURN_SETTLEMENT" request-type="SERVICE" enabled="true" >
            <user-type-path>/transactor/categoryCode</user-type-path>
            <request-type>INITIATION</request-type>
            <privilege-code>CHURN_SETTLEMENT</privilege-code>
            <validate-attributes>USER_TYPES</validate-attributes>
        </step>
        <step id="populateServiceChargesDetailsBeforePausing" name="validate.additional.parties.v2" enabled="true"/>
        <!--<step id="first.approval" name="second.party.authentication.required" enabled="true"
              removeSensitiveFieldsAfter="false">
            <onResume>
                <subSteps>
                    <subStep id="load.token.user.for.approval" name="load.token.user.for.approval" financial-service="true"/>
                    <subStep id="load.party" name="load.party.v2" removeSensitiveFieldsAfter="false"/>
                    <subStep id="validate.approver" name="validate.approver" removeSensitiveFieldsAfter="false"/>
                    <subStep id="validate.workflow.role" name="validate.workflow.role" Level="1" enabled="true" requestedParty="party"/>
                    <subStep id="txn.resume" name="txn.resume" removeSensitiveFieldsAfter="false"/>
                    <subStep id="txn.pause.reason" name="txn.pause.reason"/>
                    <subStep id="party.validation.first.approval" name="party.validation.v2"
                             removeSensitiveFieldsAfter="false">
                        <enabled>true</enabled>
                    </subStep>
                </subSteps>
            </onResume>
            <onPause>
                <subSteps>
                    <subStep id="transaction.id.generation" name="transaction.id.generation" enabled="true"
                             removeSensitiveFieldsAfter="false"/>
                    <subStep id="txn.pause" name="txn.pause" removeSensitiveFieldsAfter="false">
                        <addInfokey_1>First Name</addInfokey_1>
                        <addInfokey_2>Last Name</addInfokey_2>
                        <addInfokey_3>Email</addInfokey_3>
                        <addInfokey_4>Mobile Number</addInfokey_4>
                        <pauseLevel>1</pauseLevel>
                        <additionalInfo>4</additionalInfo>
                        <action>TI</action>
                        <addInfoValue_1>[transactor][userName]</addInfoValue_1>
                        <addInfoValue_2>[transactor][lastName]</addInfoValue_2>
                        <addInfoValue_3>[transactor][emailId]</addInfoValue_3>
                        <addInfoValue_4>[transactor][mobileNumber]</addInfoValue_4>
                        <eventOnSuccess>requestIsInitiated</eventOnSuccess>
                    </subStep>
                    <subStep id="txn.pause.reason" name="txn.pause.reason"/>
                </subSteps>
            </onPause>
            <onCancel>
                <subSteps>
                    <subStep id="load.paused.payload" name="load.paused.payload" removeSensitiveFieldsAfter="false"/>
                    <subStep id="load.token.user.for.approval" name="load.token.user.for.approval" financial-service="true"/>
                    <subStep id="load.party" name="load.party.v2" removeSensitiveFieldsAfter="false"/>
                    <subStep id="validate.approver" name="validate.approver" removeSensitiveFieldsAfter="false"/>
                    <subStep id="populate.user.workflow.role" name="populate.user.workflow.role" removeSensitiveFieldsAfter="false">
                        <requestedParty>party</requestedParty>
                        <enabled>true</enabled>
                    </subStep>
                    <subStep id="validate.workflow.role" name="validate.workflow.role" Level="1" enabled="true" requestedParty="party"/>
                    <subStep id="txn.cancel" name="txn.cancel" removeSensitiveFieldsAfter="false">
                        <routedBack>false</routedBack>
                        <initiatorCancel>true</initiatorCancel>
                        <action>TF</action>
                        <eventOnSuccess>requestIsRejectedAtLevel1</eventOnSuccess>
                    </subStep>
                    <subStep id="txn.pause.reason" name="txn.pause.reason"/>
                </subSteps>
            </onCancel>
        </step>-->
        <step id="resolve.workflow.rule" name="resolve.workflow.rule" enabled="true">
            <onResume>
                <subSteps>
                    <subStep id="load.paused.payload" name="load.paused.payload"/>
                    <subStep id="load.token.user.for.approval" name="load.token.user.for.approval"
                             financial-service="true"/>
                    <subStep id="load.party" name="load.party.v2"/>
                    <subStep id="validate.approver" name="validate.approver"/>
                    <subStep id="approve.workflow" name="approve.workflow">
                    </subStep>

                    <subStep id="txn.pause.reason" name="txn.pause.reason">            </subStep>
                </subSteps>
            </onResume>
            <onPause>
                <subSteps>
                    <subStep when="[isFreezeAmount]!=null and [isFreezeAmount]==true" id="transaction.freeze.amount"
                             name="transaction.freeze.amount"/>
                    <subStep id="txn.pause" name="txn.pause" removeSensitiveFieldsAfter="false">
                        <isNewWorkflowEngine>true</isNewWorkflowEngine>
                    </subStep>
                    <subStep id="txn.pause.reason" name="txn.pause.reason"/>
                </subSteps>
            </onPause>
            <onCancel>
                <subSteps>
                    <subStep id="load.paused.payload.cancel" name="load.paused.payload"/>
                    <subStep id="load.token.user.for.approval" name="load.token.user.for.approval"
                             financial-service="true"/>
                    <subStep id="load.party" name="load.party.v2"/>
                    <subStep id="validate.approver" name="validate.approver"/>
                    <subStep
                            when="[cancelFreezeAmount]!=null and [cancelFreezeAmount]==true"
                            id="transaction.clear.frozen.amount" name="transaction.clear.frozen.amount"/>
                    <subStep id="cancel.workflow" name="cancel.workflow">
                    </subStep>
                    <!--<subStep id="txn.pause.reason" name="txn.pause.reason">
                        <eventOnCancelFailure>orderCancelFailed</eventOnCancelFailure>
                    </subStep>
                    <subStep id="update.order" name="update.order" removeSensitiveFieldsAfter="false">
                        <order-state>VALIDATION_FAILED</order-state>
                    </subStep>-->
                </subSteps>
            </onCancel>
            <eventOnFailure>validationFailed</eventOnFailure>
            <eventOnSuccess>second.party.authentication.required</eventOnSuccess>
        </step>
        <!--<step when="[isSecondLevelApprovalRequired] == true" id="resolve.workflow.rule1" name="resolve.workflow.rule" enabled="false">
            <onResume>
                <subSteps>
                    <subStep id="load.paused.payload" name="load.paused.payload"/>
                    <subStep id="load.token.user.for.approval" name="load.token.user.for.approval"
                             financial-service="true"/>
                    <subStep id="load.party" name="load.party.v2"/>
                    <subStep id="validate.approver" name="validate.approver"/>
                    <subStep id="approve.workflow" name="approve.workflow">
                    </subStep>

                    <subStep id="txn.pause.reason" name="txn.pause.reason">            </subStep>
                </subSteps>
            </onResume>
            <onPause>
                <subSteps>
                    <subStep when="[isFreezeAmount]!=null and [isFreezeAmount]==true" id="transaction.freeze.amount"
                             name="transaction.freeze.amount"/>
                    <subStep id="txn.pause" name="txn.pause" removeSensitiveFieldsAfter="false">
                        <isNewWorkflowEngine>true</isNewWorkflowEngine>
                    </subStep>
                    <subStep id="txn.pause.reason" name="txn.pause.reason"/>
                </subSteps>
            </onPause>
            <onCancel>
                <subSteps>
                    <subStep id="load.paused.payload.cancel" name="load.paused.payload"/>
                    <subStep id="load.token.user.for.approval" name="load.token.user.for.approval"
                             financial-service="true"/>
                    <subStep id="load.party" name="load.party.v2"/>
                    <subStep id="validate.approver" name="validate.approver"/>
                    <subStep
                            when="[cancelFreezeAmount]!=null and [cancelFreezeAmount]==true"
                            id="transaction.clear.frozen.amount" name="transaction.clear.frozen.amount"/>
                    <subStep id="cancel.workflow" name="cancel.workflow">
                    </subStep>
                    <subStep id="txn.pause.reason" name="txn.pause.reason">
                        <eventOnCancelFailure>orderCancelFailed</eventOnCancelFailure>
                    </subStep>
                    <subStep id="update.order" name="update.order" removeSensitiveFieldsAfter="false">
                        <order-state>VALIDATION_FAILED</order-state>
                    </subStep>
                </subSteps>
            </onCancel>
            <eventOnFailure>validationFailed</eventOnFailure>
            <eventOnSuccess>second.party.authentication.required</eventOnSuccess>
        </step>-->
        <step id="transaction.posting" name="transaction.posting" enabled="true" removeSensitiveFieldsAfter="false">
            <action>TS</action>
            <eventOnSuccess>requestIsApprovedAtLastLevel</eventOnSuccess>
        </step>
    </steps>
</serviceFlow>
