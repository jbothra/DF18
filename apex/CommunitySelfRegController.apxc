public class CommunitySelfRegController {
    @AuraEnabled
    public static String createExternalUser(
        String username, String password, String startUrl, String fname, 
        String lname,String[] products, Boolean hasOptedTracking, Boolean hasOptedSolicit) {
            Savepoint sp = null;
            try {
                sp = Database.setsavepoint();
                system.debug(sp);
                
                // creating an user object. 
                User u = new User();
                u.Username = username;
                u.Email = username;
                u.FirstName = fname;
                u.LastName = lname;

                // Default ui on mobile is set to S1 for user created using site object. 
                // Enable this perm to change it to community.
                u.UserPreferencesHideS1BrowserUI = true;
                
                // generating unique value for community nickname. 
				String nickname = ((fname != null && fname.length() > 0) ? fname.substring(0,1) : '' ) + lname.substring(0,1);
            	nickname += String.valueOf(Crypto.getRandomInteger()).substring(1,7);
                u.CommunityNickname = nickname;
                
                System.debug('creating user');
                
                // Creating portal user. 
                // passing in null account id forces it to read this form network setting (set using community workspace)
                String userId = Site.createPortalUser(u, null, password);
                
                // Setting concent selection values. 
                // For this, GDPR (Individual, Consent management) needs to be enabled on the org. 
                Individual ind = new Individual();
                ind.LastName = lname;
                ind.HasOptedOutSolicit = !hasOptedSolicit;
                ind.HasOptedOutTracking = !hasOptedTracking;
                insert(ind);
                
                // other information of the contact can be updated here. 
                Contact contact = new Contact();
                contact.Id = u.ContactId;
                contact.IndividualId = ind.Id;
                update(contact);       
                
                // creating asset
                createAssets(u, products, lname);
                                                
                // return login url. 
                if (userId != null && password != null && password.length() > 1) {  
                    ApexPages.PageReference lgn = Site.login(username, password, startUrl);
                    return lgn.getUrl();
                }
                
            }
            catch (Exception ex) {
                Database.rollback(sp);
                System.debug(ex.getMessage());
                return null;
            }
            return null; 
    }
    
    private static void createAssets(User u, String[] products, String last_name) {
        if(products == null) {
            return; 
        }
        Contact c = [SELECT firstName,lastName,accountId FROM Contact WHERE Id = :u.ContactId];
        List<Product2> prdts = [SELECT id, name FROM Product2 WHERE Id IN :products]; 
        for(Product2 product : prdts){
            // creating assset
            Asset asset = new Asset();
            asset.Name = product.name;
            asset.AccountId = u.AccountId;
            asset.ContactId = u.ContactId;
            asset.Product2Id = product.id;
            insert(asset);
        }
    }
}