From: https://github.com/distributed-text-services/workshops/issues/18

# Proponent / Mentor
Robert Casties (@robcast)

# Brief Description
The International Image Interoperability Framework (IIIF, https://iiif.io) is a set of specifications and a community revolutionizing access to images on the internet. IIIF viewers are used by many cultural heritage institutions to present scanned books and manuscripts but the integration of full texts e.g. for searchable overlays, text+image views or richer interactive editions is still rather weak. It would be great if the DTS API could fill this void and become the IIIF analog for text use in the internet.

I would like to look at the integration points of the IIIF API with respect to full-text and the DTS API with respect to images on the specification level, collect concrete use cases in both communities, evaluate problem areas in concrete application use, and develop extensions to the specifications if necessary. This could be mostly done at the specification/paper level but I would also like to look at concrete applications like full-text extensions to the Mirador IIIF viewer.

# Coding skills needed
Interest in reading and discussing specification prose would be very welcome. Coding skills would not be needed unless somebody would offer to create a prototype using Mirador (Javascript/React) or other IIIF viewer.

# Team members (if applicable)
The hack is open to any and all interested in bringing IIIF and DTS together.

----

# Use case 1: Parallel pages

As a scholar I want to see the transcription of a page from the textual edition next to the scanned image of the original page so that the transcription helps me reading the original document or to check the transcription in the textual edition with an image of the original.

For a UX example see http://telota.bbaw.de/kant_op/edition.html#/C05/005 (Online version of Immanuel Kant's Opus Postumum by BBAW)
![viewer-kant-op](https://user-images.githubusercontent.com/1488847/135756628-91c46bae-e0ba-44e3-aab9-1b205b303899.png)

----

# Design goal

Using IIIF and DTS it should be possible to provide a full featured viewer as a web application that is fully independent from both the (IIIF) content server for the images and the (DTS) content server for the text.

This means that it should be possible to create a new presentation using a text from a DTS provider that does not know about any images and images from an IIIF provider that does not know about any text, possibly creating new DTS Navigation endpoints or IIIF manifests in the process.

Alternatively the TEI document already references a IIIF Canvas or Image Service for each page or the IIIF Manifest already references a TEI fragment for each Canvas. But this case requires that the authors of the TEI document have exact knowledge of the page image URIs when creating the document which is an practical limitation, and/or it requires that the document is updated every time the image URIs change.

----

# Questions

- Given a DTS Navigation endpoint how do I request a list of all logical page references for a document?
  - TODO: look at citation systems (annotation system?)
- Given a DTS document endpoint how do I request the fragment that corresponds to a logical page?
- Given a TEI text, how do I encode the fragment that corresponds to a logical page?
  - `<pb n=XX facs=XX/>`
- Given a IIIF Manifest how do I find the Canvas that corresponds to a logical page?
- Given a IIIF Manifest how do I find the text version corresponding to a given Canvas (image)?
- How do I link the logical page in TEI/DTS to the Canvas URI in IIIF?
- Which is the leading document that the viewer application reads first to discover the other? IIIF Manifest or DTS Navigation Endpoint?

----

# Use case 2: Parallel sections

As a scholar I want to display a section of a text like a paragraph or an inscription next to an image showing the corresponding part of an image of the original object so that I can use the text to help deciphering the image or use the image to check the transcription in the text.

----

# Use case 2a: Text search and highlighting on the image

(special case of Use case 2)

As a scholar I want to to do a full text search and see results highlighted on a scanned image and I want to select words on a scanned page with the mouse that return the selected text so that I can quickly find a page of a scanned book and copy a text passage into a separate document.

This is an example of selecting text on the image
![screen-bsb-text-highlight](https://user-images.githubusercontent.com/1488847/135856555-b22c23fa-13f5-4120-a5a0-bf68f3b3094b.png)

For a full text search see https://www.digitale-sammlungen.de/en/view/bsb11278642?page=131&q=siegfried

----

Pietro Liuzzo
> > # Use case 1: Parallel pages
> > As a scholar I want to see the transcription of a page from the textual edition next to the scanned image of the original page so that the transcription helps me reading the original document or to check the transcription in the textual edition with an image of the original.
> 
> Hi! I think I do this in my implementation by extending members of the navigation API.
> 
> ```json
> "member" : [ {
>     "dts:citeType" : "folio",
>     "dts:dublincore" : {
>       "dc:source" : [ {
>         "@type" : "sc:Canvas",
>         "@id" : "https://betamasaheft.eu/api/iiif/ESum035/canvas/p1"
>       }, {
>         "@type" : "sc:Canvas",
>         "@id" : "https://betamasaheft.eu/api/iiif/ESum035/canvas/p2"
>       } ]
>     },
>     "dts:ref" : "1"
>   }
> ```
> 
> for each ref in the navigation response I compute and provide the relevant canvases available. in cases where we have a lot of manuscripts with images, but with little precision in their encoding, we have things like the following, returning a pointer to the entire manifest https://betamasaheft.eu/api/dts/navigation?id=https://betamasaheft.eu/LIT1546Genesi where we know a bit more precisely then the link to the relevant manifests can be more precise, e.g. https://betamasaheft.eu/api/dts/navigation?id=https://betamasaheft.eu/LIT4035SenkessarDL
> 
> I both cases we have to provide for multiple images. in the first case the images of 1 folio will be 2 since photos are of openinings. In the further examples the images will be all sets which we know about and for which we can build an iiif uri, most of the time more then one.

----

Pietro Liuzzo
> from the navigation response to https://betamasaheft.eu/api/dts/navigation?id=https://betamasaheft.eu/LIT4035SenkessarDL you can then use e.g. "dts:passage" : "/api/dts/document?id=https://betamasaheft.eu/LIT4035SenkessarDL" add the ref you want among those available and add it up to retrieve the text e.g. /api/dts/document?id=https://betamasaheft.eu/LIT4035SenkessarDL&ref=FirstHalf
> 
> so from the response you have all available refs, which you can use to query the document API and the links to relevant images via IIIF

----

Pietro Liuzzo
> > # Use case 2a: Text search and highlighting on the image
> > (special case of Use case 2)
> > As a scholar I want to to do a full text search and see results highlighted on a scanned image and I want to select words on a scanned page with the mouse that return the selected text so that I can quickly find a page of a scanned book and copy a text passage into a separate document.
> 
> Personally, I am not sure this is a use case served by DTS. There is no search / query API at the moment. as an alternative Transkribus may be more helpful in this direction?

----

Pietro Liuzzo
> > * Given a TEI text, how do I encode the fragment that corresponds to a logical page?
> 
> using the ref. the document API should be able to resolve to the correct passage for a given combination of ref/range/level parameters
> 
> > * Given a IIIF Manifest how do I find the Canvas that corresponds to a logical page?
> 
> I do this looking up the TEI and reproducing the canvas `@id` and relying on the fact that I know how those are built.
> 
> > * Which is the leading document that the viewer application reads first to discover the other? IIIF Manifest or DTS Navigation Endpoint?
> 
> I would definitely opt for the DTS endpoint.

----
Notes

see also http://jeffreycwitt.com/IIIFWorkshop/docs/doc4

