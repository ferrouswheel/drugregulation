Rationalised Drugs
==================

This project is a technical proposal for implementing drug rationing in a world where
recreational drugs are treated as a health concern and not a criminal charge.

## Rationale

The code herein consists (or will consist) of a system to track drug
provisioning without risking the privacy of individuals who choose to consume
them. The core functionality of this research is to limit the frequency of drug
purchase, and avoid dangerous over consumption.

Take the following example. In this hypothetical world where all drugs are
regulated, it might be fine to buy a pill of MDMA every week, and more than
that could be deemed overuse.  With alcohol, most countries have the law that
bars are not allowed to serve intoxicated persons in order to limit binge
drinking. However, pills and other substances are harder to control in this way
as one could simply buy many at once and consume them before the effects are
noticable to a salesperson. One proposal is to have drugs rationed so that every
individual would be allowed a predefined maximum frequency of use.

In order to prevent individuals running from one store to the next to buy more
of a rationed substance, some kind of tracking is needed. Ration cards and
stamps were an approach used in war and other emergency situations, however
these are prone to being traded amongst individuals creating another black
market which is what we wish to avoid.

Since this is the 21st century, another alternative is an online system that
validates whether a buyer is allowed to complete a transaction. A store would
enter the customer's details (e.g a driver's license) and the system would
return a simple yes/no. A major issue with this approach is that there would
then exist a repository of people's drug habits. Given that social change
generally takes a long time, it is unlikely that the general publics prejudices
against drug use will disappear even if legislation changes to regulate it.
Thus the database of drug habits would likely be of interest to insurance
companies and employers. While best-practices in security could be put in place
to try to avoid this data falling in the wrong hands, it'd be better to have
a system that is built to keep the details somewhat anonymous but still
functional. 

To summarise, the envisioned process would look like:

1. Consumer wishes to purchase drug A.
2. Retailer enters consumer's ID and drug type.
3. If the system returns "yes" the transaction can proceed, if "no" the user
can be told whether they have personally had their right removed (due to
criminal convictions while intoxicated) or due to having used said drug too
recently.

## Background

Imagine if you will, a world where recreational substances are regulated and no
longer contribute to a global war between violent gangs, overfunded government
organisations, and increased risk to recreational substance users. I prefer to use
"recreational substances" as opposed to the colloqual "drugs", since many things
are drugs. Coffee, alcohol, prescription medicine are drugs, but this fact
is conveniently forgotten by media, the general public, and policy makers.
Indeed, many foods have psychoactive substances in them too (chocolate, turkey,
chilli, etc.). The dividing line between what you can legally consume and that
which you will be punished for is often arbitary.

"But some drugs are bad!" I hear you say. Well, comparatively, alcohol is one
of the [worst drugs for society and the individual][1]
and the relative standing of various drugs is summarised in the following
[graph][2]:

![2](http://download.thelancet.com/images/journalimages/0140-6736/PIIS0140673610614626.gr2.lrg.jpg)

A bit different to what the media, your government or drug enforcement agencies
would have you believe.

Since this project suggests a rationing approach, an acceptable usage level could be
defined. Overuse could be based on health evidence, rather than puritanical presumption
that drugs cause health risks that are greater than other substances or
recreational activities ([horse riding anyone?][3]).

Recently in New Zealand, legislation has moved to block various "legal highs"
from being sold, and due the speed at which new psychoactive substances are
being discovered and produced (indeed, the number of chemical variations that
will produce a recreational effect is very large) an analogues act was passed.
I'm not right up with the latest in the legislation, but one interesting
comment from people that are is that there was a move to put the onus of
proof, for safety of consumption, on the sellers. However this opens up the
question of what happens when drugs that are currently illegal pass these
safety standards, and other drugs that are legal, don't. (TODO: expand and link
to relevent legislation).

[1]: http://www.thelancet.com/journals/lancet/article/PIIS0140673610614626/abstract "Drug harms in the UK: a multicriteria decision analysis")
[2]: http://download.thelancet.com/images/journalimages/0140-6736/PIIS0140673610614626.gr2.lrg.jpg "Comparative normalised drug harms"
[3]: http://www.ncbi.nlm.nih.gov/pubmed/19158127 "Equasy -- an overlooked addiction with implications for the current debate on drug harms. Journal of Psychopharmacology 23 (1): 3–5."

## Technical

**Disclosure**: while I'm reasonably technically savvy, I am not a crytogeek, and
I'm very aware that getting cryptography processes _right_ is hard. Thus
I present it as a prototype for comment rather than a functional system.

There are several questions we need to answer:

* **Identification of individuals**. In the simplest case, everyone uses a driver's
  license, but not everyone has one of these and an exclusionary system is not
  ideal. The problem with allowing multiple identification techniques is 
  resolving these identities to one individual.
* **Allowed frequency** - for a given substance, what is the allowable
  frequency of use and dosage size. The actual decision is health decision, but
  the system needs to account for multiple timeframes/quantities.
* **Blacklisted individuals** - how do we determine whether an individual is
  allowed to be using recreational substances.
* **Retailer accountability** - If the supply of a substance is rationed, then retailers
  need to be accountable for the substances they sell. Thus distributed batches
  of goods need to match with the number of a transactions to legitimate
  buyers. Without this, the retailer becomes the weakest link and potential for
  the black market over supplying individuals.
* **Transaction authorisation** - When an individual attempts to purchase
  through a retailer, the retailer gets told whether they are allowed to
  complete the purchase.
* **Protection of historical use** - a history of transactions per individual
  is needed in order to restrict frequency of use, however, this history could
  be harmful if linked to an individual.

For each transaction, we have:

* `user_id` - In order to resolve identities (to avoid people using different IDs
  to avoid rationing), the simplest approach would be to have people need to
  sign up and get a unique ID for the programme... in order to get this they need
  to present some form of government recognised identification.
* `transaction_date`, `transaction_time` - the date and time of the purchase.
* `substance` - a string identifier of the substance (must be validated)

The rough idea is to use a key value store, where:

* The key is the hashed combination of `user_id` and `date_salt`. The `date_salt` is
  a random value generated daily by the system. Another table keeps track of
  the `date` -> `date_salt` mapping, and expires entries after the maximum
  "allowed frequency" across all rationed substances. This means past
  information is no longer accessable. If necessary, the obsolete `date_salt`s
  could be archived for historical analysis, but would not be accessable in
  a live system. Historical analysis would be useful to get a better
  understanding of drug use across a population, but the suggested solution
  would need to be augmented to be useful and anonymise `user_id`.
* The value is a dictionary/hash of substances purchased on that day with the
  amount purchased (each substance would have a defined unit of measure used
  consistently through out the system).

To check whether the transaction is allowed to proceed:

* Get the substance's "allowed frequency", this is the number of past days to
  check (and the number of keys we need to lookup).
* For each day work out the key to check (combine `user_id` and `date_salt`),
  find the amount of substance in question that was purchased. sthis to 

## Questions? Comments?

Please feel free to submit issues or contact me about this.

There are probably many shortcomings in my approach, so I'd love feedback.

