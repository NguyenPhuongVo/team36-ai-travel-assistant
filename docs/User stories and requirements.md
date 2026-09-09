Project: AI travel assistant for accessible aviation – team 36 (team b) 

We now know from sprint one we are enhancing an already developed chatbot rather than developing our own. We will also be limited to utilizing only the client’s own trusted AI API and database. This constraint influences all of the areas described below so therefore the stories were created with this in consideration and ranked using the MoSCoW method to help us decide what we should build in Sprint 2. 
 
Prior to beginning any development, we need to comprehend how the AI API and database operate including authorization methods, response formats, available data, limits of the available data. The client has stated that this must occur prior to implementation and not simultaneously. 

Must-have—core MVP 
 
Us-1: Communicate accessibility needs as a disabled traveller. I want to inform the chatbot about my accessibility needs so the information returned relates specifically to me.  
• You may communicate your needs via free text or utilize the guided prompts provided by the system and the system shall not make assumptions regarding your needs based upon your input.  
• The first time you provide this information you should not need to re-enter this information throughout the remainder of the session. 

Us-2: obtain accessible information regarding my journey as a PRM traveller. I want travel information which reflects my requirements and allows me to rely upon the information.  
• All the information will originate from the client's api/database/trusted sources — none of which will be obtained from untrusted third party sources.  
• The information that is relevant to your inquiry will include labels indicating relevance; conversely if the system cannot find any relevant information, the system will indicate that it was unable to locate any relevant information as opposed to providing you with non-relevant information. 

Us-3: Obtain accessible accommodation information. As a PRM traveller I want the accommodation information presented in the same manner as my travel information, so I am able to plan both aspects of my journey.  
• Similarly, all of the accommodation information will come from the same trusted source; the same format used for presenting travel information; consistent presentation of both types of information.  
• Accommodations listed will contain detailed descriptions of their accessibility features they will never be implied and/or left for you to infer. 

Us-4: Understand why something was presented to me as relevant. As a PRM traveller I want to understand why certain information is relevant or not to enable me to avoid having to interpret the information on my own.  
• Why something is relevant will be indicated clearly not buried within paragraphs of other information — presented in plain language without technical/jargon terminology. 

Us-5: View Supporting media alongside responses. As a PRM traveller I would like images/links Supporting answers to appear along-side them, enabling me to confirm whether or not I believe I can take action upon the advice/information being given.  
• Supporting media will display inline where applicable; if not applicable then I do not see blank placeholders. 

US-6: Consistent interaction flow. As a PRM traveller I want the responses to the questions i ask to consistently present themselves in the same manner regardless of the question I ask, so I don’t have to constantly learn new ways to interact with the tool for every question i ask.  
• the layout will remain consistent for each response; receiving necessary information will require fewer clicks than possible. 

 

Should Be Done Before We Can Call It an MVP  
The ability to easily compare options as a PRM traveller, instead of having to read them over again. The results of a search should look the same, have the same fields with the same labels etc. for that kind of thing. 

Accessibility information should be easy to get to. No more than two clicks/taps between you and critical details like whether the wheelchair accessible toilet is working. 

Should Be Done Eventually, But Not Now 
Things that the customer has expressed interest in some day but are not interested enough to do now. These will likely impact our architecture, but we won’t build these into the product until we know more about what the customer wants.  
US-9: Flight Radar integration. This could potentially be done soon if it seems feasible to add after doing the technical investigation.  
US-10: Additional travel service integrations. The client needs to tell us which services would be good for us to integrate. Once we know this, we can plan how we’re going to make sure we’ve got an open architecture to allow us to easily add new ones.  
US-11: Multi-language support. We can’t implement multi-language support until we know what languages we’ll need to translate. We’d like to investigate this sooner rather than later, so we don’t lose time when we finally get the green light to move forward.  
US-12: Faster response times (targeted at around 5 – 7 seconds). We can benchmark where we stand today once we complete the technical investigation, but it’s too big of a stretch for us to commit to it in Sprint 2. 

What Should Happen in Sprint 2 
First, let’s do the technical investigation. Everything else depends on the findings of this investigation. Next, build US-1 through US-6. These are the majority of the pieces needed to create the MVP and where most of our development effort should be focused. 
If we have time remaining after completing US-1 through US-6, then try to also implement US-7 and US-8. Lastly, leave US-9 through US-12 alone for now. They are clearly defined as future work items. 

 
